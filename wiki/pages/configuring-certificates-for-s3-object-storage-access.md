---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L794-830
verified: 2026-08-25
confidence: documented
---

# Configuring certificates for S3-compatible object storage access

Adds the in-cluster API server's CA certificate to RHOAI's trusted CA
configuration ([[dscinitialization]]'s `customCABundle`), so RHOAI
components can securely reach an in-cluster object storage endpoint or
database that serves TLS with an internal/cluster-signed certificate.

**Only needed when**: the object storage/database endpoint uses TLS signed
by an internal or cluster CA. **Not needed when**: the endpoint is plain
HTTP, or its certificate is already covered by the cluster-wide CA bundle
(see [[adding-certificates-to-cluster-wide-ca-bundle]]).

## Preconditions

- Cluster administrator access.
- OpenShift CLI (`oc`) installed.
- An object storage solution or database already deployed in the cluster.

## Steps

1. Log in: `oc login api.<cluster_name>.<cluster_domain>:6443 --web`
2. Preserve the existing custom CA bundle to a file (so later steps append
   rather than replace):
   ```
   oc get dscinitializations.dscinitialization.opendatahub.io default-dsci -o json \
     | jq -r '.spec.trustedCABundle.customCABundle' > /tmp/my-custom-ca-bundles.crt
   ```
3. Append every namespace's identical `kube-root-ca.crt` config map (the
   internal API server's CA) to that file:
   ```
   oc get configmap kube-root-ca.crt -o jsonpath="{['data']['ca\.crt']}" >> /tmp/my-custom-ca-bundles.crt
   ```
4. Patch the combined bundle back onto the DSCI:
   ```
   oc patch dscinitialization default-dsci --type='json' \
     -p='[{"op":"replace","path":"/spec/trustedCABundle/customCABundle","value":"'"$(awk '{printf "%s\\n", $0}' /tmp/my-custom-ca-bundles.crt)"'"}]'
   ```

## Verification

- From a workbench pod that mounts the updated bundle, confirm the TLS
  handshake succeeds:
  ```
  oc exec -n <namespace> <workbench-pod> -- \
    curl -sS -o /dev/null -w "%{http_code}\n" https://<s3-endpoint>
  ```
  Any HTTP status code (`200`, `403`, ...) means the handshake succeeded. A
  `curl` self-signed-certificate or verification-failure error means the
  issuing CA is still missing from the bundle.
- A component configured against the in-cluster storage/database (e.g. a
  pipeline server using an in-cluster database) reaches a running state.
