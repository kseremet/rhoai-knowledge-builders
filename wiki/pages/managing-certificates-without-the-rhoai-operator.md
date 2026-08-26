---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1130-1157
verified: 2026-08-25
confidence: documented
---

# Managing certificates without the RHOAI Operator

Switches `spec.trustedCABundle.managementState` on the [[dscinitialization]]
DSCI object from `Managed` to `Unmanaged`, so the [[rhoai-operator]] stops
auto-updating the `odh-trusted-ca-bundle` ConfigMap — for organizations
that manage trusted CA bundles independently (e.g. via Ansible). See
[[how-rhoai-handles-certificates]] for what each management state means.

## Preconditions

- Cluster administrator access.
- OpenShift CLI (`oc`) installed.

## Steps

1. Installed Operators → **Red Hat OpenShift AI Operator** → **DSC
   Initialization** tab → `default-dsci` → **YAML** tab.
2. Change `trustedCABundle.managementState` to `Unmanaged`:
   ```yaml
   spec:
     trustedCABundle:
       managementState: Unmanaged
   ```
3. **Save**. This prevents automatic updates when `customCABundle`
   changes, but does **not** remove the existing `odh-trusted-ca-bundle`
   ConfigMap.

## Verification

1. Set `customCABundle` to a test value and save:
   ```yaml
   spec:
     trustedCABundle:
       managementState: Unmanaged
       customCABundle: example123
   ```
2. Confirm the test value does **not** propagate:
   `oc get configmap odh-trusted-ca-bundle -n <namespace> -o jsonpath={.data.odh-ca-bundle\.crt}`
   — output should not contain `example123`, since the Operator no longer
   reconciles the bundle while `Unmanaged`.

## To go back

Set `managementState` back to `Managed` to resume automatic reconciliation.
