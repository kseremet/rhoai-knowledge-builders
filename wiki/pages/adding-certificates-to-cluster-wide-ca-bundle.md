---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L702-738
verified: 2026-08-25
confidence: documented
---

# Adding certificates to a cluster-wide CA bundle

Adds a self-signed certificate to the OpenShift cluster-wide CA bundle
(`ca-bundle.crt`), which the Cluster Network Operator then injects into
[[rhoai-operator]]'s `odh-trusted-ca-bundle` ConfigMap automatically — see
[[how-rhoai-handles-certificates]] for the mechanism. Choose this over
[[adding-certificates-to-custom-ca-bundle]] when the certificate needs to
be trusted across multiple services or cluster-wide by policy.

## Preconditions

- A self-signed certificate created and saved to a file (e.g. via OpenSSL),
  for example `example-ca.crt`.
- Cluster administrator access.
- OpenShift CLI (`oc`) installed.

## Note

By default `spec.trustedCABundle.managementState` on the [[dscinitialization]]
DSCI object is `Managed`. If it's `Unmanaged`, the `odh-trusted-ca-bundle`
ConfigMap must be updated manually with the new cluster-wide bundle instead
— see [[managing-certificates-without-the-rhoai-operator]].

## Steps

1. Create a ConfigMap with the root CA certificate, in the
   `openshift-config` namespace:
   ```
   oc create configmap custom-ca \
     --from-file=ca-bundle.crt=</path/to/example-ca.crt> \
     -n openshift-config
   ```
2. Point the cluster-wide proxy configuration at it:
   ```
   oc patch proxy/cluster \
     --type=merge \
     --patch='{"spec":{"trustedCA":{"name":"custom-ca"}}}'
   ```

## Verification

```
oc get configmaps --all-namespaces -l app.kubernetes.io/part-of=opendatahub-operator | grep odh-trusted-ca-bundle
```
Confirms all non-reserved namespaces have the ConfigMap.
