---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L2027-2062
verified: 2026-08-25
confidence: documented
---

# Enabling the observability dashboard in the UI

**Technology Preview, as of 3.5.** Surfaces the **Observe & monitor** →
**Dashboard** menu item in the RHOAI dashboard, once
[[enabling-observability-stack|the observability stack]] itself is already
enabled on the [[dscinitialization]] CR.

## Preconditions

- RHOAI administrator privileges.
- The observability stack already enabled — see
  [[enabling-observability-stack]].

## Steps

1. Log in to the OpenShift console as an RHOAI administrator.
2. **Administrator** perspective → **Home** → **API Explorer** → search
   `OdhDashboardConfig` → open the CR.
3. Project `redhat-ods-applications` → **Instances** →
   `odh-dashboard-config` instance → **YAML**.
4. Under `spec.dashboardConfig`, add/update:
   ```yaml
   spec:
     dashboardConfig:
       observabilityDashboard: true
   ```
5. **Save**, then **Reload** to sync the change.

## Verification

Log in to the RHOAI dashboard and confirm **Observe & monitor** →
**Dashboard** appears in navigation.
