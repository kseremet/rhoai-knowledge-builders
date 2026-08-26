---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1169-1202
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1203-1224
verified: 2026-08-25
confidence: documented
---

# Removing the CA bundle

Overrides [[rhoai-operator]]'s default certificate handling (see
[[how-rhoai-handles-certificates]]) by removing the `odh-trusted-ca-bundle`
ConfigMap — either everywhere, or from one namespace. Use this if
implementing a different authentication approach for RHOAI.

## Removing from all namespaces

Sets `spec.trustedCABundle.managementState` on the [[dscinitialization]]
DSCI object to `Removed`.

**Preconditions**: cluster administrator access; OpenShift CLI installed.

**Steps**:
1. Installed Operators → **Red Hat OpenShift AI** → **DSC Initialization**
   tab → `default-dsci` → **YAML**.
2. Set:
   ```yaml
   spec:
     trustedCABundle:
       managementState: Removed
   ```
3. **Save**.

**Note**: the ConfigMap is only deleted from namespaces by setting this
field to `Removed` — deleting the DSCInitialization object itself does
**not** delete the config maps.

**Verification**:
- `oc get configmaps --all-namespaces | grep odh-trusted-ca-bundle` —
  should return nothing.
- Confirm no new-namespace injection: `oc create namespace <test-namespace>`
  then `oc get configmap odh-trusted-ca-bundle -n <test-namespace>` — `oc
  get` reports not found.
- `oc delete namespace <test-namespace>` when done testing.

## Removing from a single namespace

Annotates just that namespace, leaving `trustedCABundle.managementState`
untouched cluster-wide.

**Preconditions**: cluster administrator access; OpenShift CLI installed.

**Steps**:
```
oc annotate ns example-namespace security.opendatahub.io/inject-trusted-ca-bundle=false
```

**Verification**:
- `oc get configmap odh-trusted-ca-bundle -n example-namespace` — returns
  `configmaps "odh-trusted-ca-bundle" not found`.
- To restore it, remove the annotation:
  ```
  oc annotate ns example-namespace security.opendatahub.io/inject-trusted-ca-bundle-
  oc get configmap odh-trusted-ca-bundle -n example-namespace
  ```
  — the Operator recreates the config map.
