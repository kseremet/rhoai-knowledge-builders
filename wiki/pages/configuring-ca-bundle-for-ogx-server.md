---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L951-989
verified: 2026-08-25
confidence: documented
---

# Configuring a CA bundle for OGX

Configures an [[ogx-server]] (`OGXServer` CR) to trust a self-signed or
private CA, using the recommended pattern of referencing
`odh-trusted-ca-bundle` — the config map RHOAI already maintains in every
non-reserved namespace (see [[how-rhoai-handles-certificates]]). For
dedicated or cross-namespace config maps instead, see
[[ogx-server-ca-bundle-reference]].

**Important**: completing this restarts the OGX server pod — plan for a
brief interruption on a production-serving instance.

## Preconditions

- OpenShift 4.19 or later.
- Logged in to RHOAI, cluster administrator privileges.
- The OGX Operator activated (see "Activating the OGX Operator", not yet
  ingested).
- At least one `OGXServer` instance already deployed (see "Deploying a OGX
  server", not yet ingested).
- Required CAs already added to the cluster-wide or a custom CA bundle
  (see [[adding-certificates-to-cluster-wide-ca-bundle]] /
  [[adding-certificates-to-custom-ca-bundle]]).
- OpenShift CLI (`oc`) installed.

## Steps

1. Log in to the RHOAI web console as a cluster administrator.
2. Select the project containing your `OGXServer` CR.
3. Confirm `odh-trusted-ca-bundle` is present (Workloads → ConfigMaps) and
   contains both `ca-bundle.crt` (cluster-wide CA bundle, injected by CNO)
   and `odh-ca-bundle.crt` (custom CAs, from the DSCI's `customCABundle`).
4. Reference it from the `OGXServer` CR: Home → Search → `OGXServer` (not
   `OGX`, which is a separate internal RHOAI resource) → select your
   instance → **YAML**:
   ```yaml
   apiVersion: ogx.io/v1beta1
   kind: OGXServer
   metadata:
     name: my-ogx
     namespace: my-ogx-namespace
   spec:
     distribution:
       name: rh-dev
     workload:
       replicas: 1
     tlsConfig:
       caBundle:
         configMapName: odh-trusted-ca-bundle
         configMapKeys:
           - ca-bundle.crt
           - odh-ca-bundle.crt
   ```
5. **Save** — the OGX server pod redeploys with the updated bundle.

## Verification

1. `oc login --token=<token> --server=<openshift_cluster_url>`
2. `oc get configmap <instance-name>-ca-bundle -n <namespace>` — the
   managed config map exists.
3. `oc get ogxserver <instance-name> -n <namespace> -o yaml` — in
   `status.conditions`, `DeploymentReady` is `status: "True"` and no
   condition reports a CA bundle validation failure in `message`.
4. `oc get pods -n <namespace> -l app.kubernetes.io/instance=<instance-name>`
   — the server pod is running.
5. `oc exec -n <namespace> <pod-name> -- ls -l /etc/ssl/certs/ca-bundle/ca-bundle.crt`
   and `oc exec -n <namespace> <pod-name> -- printenv SSL_CERT_FILE` — the
   file exists and `SSL_CERT_FILE` points to it.
6. `oc exec -n <namespace> <pod-name> -- head -20 /etc/ssl/certs/ca-bundle/ca-bundle.crt`
   — output starts with `-----BEGIN CERTIFICATE-----`.
7. `oc exec -n <namespace> <pod-name> -- curl -sS -o /dev/null -w "%{http_code}\n" <external-endpoint-url>`
   — a successful status code (e.g. `200`) confirms the chain validates. A
   self-signed-certificate/verification-failure `curl` error means the
   bundle is missing the correct issuing CA.
