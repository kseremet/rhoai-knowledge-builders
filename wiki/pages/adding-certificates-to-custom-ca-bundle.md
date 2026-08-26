---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L739-787
verified: 2026-08-25
confidence: documented
---

# Adding certificates to a custom CA bundle

Adds self-signed certificates to a custom CA bundle kept separate from the
OpenShift cluster-wide bundle, via the [[dscinitialization]] DSCI object's
`customCABundle` field. Choose this over
[[adding-certificates-to-cluster-wide-ca-bundle]] when only specific
services need the certificate (e.g. AI pipelines reaching S3-compatible
object storage over TLS) and you want to avoid a global config change.

## Preconditions

- A self-signed certificate created and saved to a file (e.g.
  `example-ca.crt`).
- Cluster administrator access.
- OpenShift CLI (`oc`) installed.

## Steps

1. Log in to the web console.
2. Installed Operators → **Red Hat OpenShift AI** → **DSC Initialization**
   tab → `default-dsci` → **YAML** tab.
3. Add the certificate under `spec.trustedCABundle.customCABundle`:
   ```yaml
   spec:
     trustedCABundle:
       managementState: Managed
       customCABundle: |
         -----BEGIN CERTIFICATE-----
         examplebundle123
         -----END CERTIFICATE-----
   ```
4. **Save**.

The Operator automatically updates the `odh-trusted-ca-bundle` ConfigMap,
adding an `odh-ca-bundle.crt` key alongside the existing `ca-bundle.crt`
key:

```yaml
data:
  ca-bundle.crt: |
    <BUNDLE OF CLUSTER-WIDE CERTIFICATES>
  odh-ca-bundle.crt: |
    <BUNDLE OF CUSTOM CERTIFICATES>
```

## Verification

```
oc get configmap odh-trusted-ca-bundle -n example-namespace -o yaml | grep examplebundle123
```
Confirms a non-reserved namespace's ConfigMap contains the custom bundle
value.
