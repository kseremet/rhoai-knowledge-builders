---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1227-1266
verified: 2026-08-25
confidence: documented
---

# Migrating to the Red Hat build of Kueue Operator

**As of 2.24**, the embedded Kueue component is deprecated in favor of the
separately-installed Red Hat build of Kueue Operator (see
[[kueue-workload-management]]). RHOAI does not auto-migrate workloads, and
running both the embedded Kueue and this Operator on the same cluster is
unsupported. Skip this procedure if
`spec.components.kueue.managementState` on the [[datasciencecluster]] CR
is already `Removed` or `Unmanaged` — only `Managed` (the embedded-Kueue
state) requires migration.

## Preconditions

- `spec.components.kueue.managementState` on the DSC is `Managed`.
- Cluster administrator privileges.
- OpenShift 4.19 or later.
- **cert-manager Operator for Red Hat OpenShift** installed and configured.

## Steps

1. Optional — the RHOAI Operator automatically moves your existing Kueue
   config from the `kueue-manager-config` ConfigMap to the new `Kueue` CR
   during migration; to keep the old ConfigMap around for reference anyway:
   ```
   oc annotate configmap kueue-manager-config -n <applications-namespace> opendatahub.io/managed=false
   ```
   (`<applications-namespace>` defaults to `redhat-ods-applications`.)
2. Log in to the OpenShift web console as a cluster administrator.
3. Uninstall the embedded Kueue component (recommended, though skippable
   if workloads must keep running uninterrupted — skipping risks temporary
   config conflicts during the RHOAI upgrade):
   - Installed Operators (**Ecosystem** → **Installed Operators** on
     OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19) →
     Red Hat OpenShift AI Operator → **Data Science Cluster** tab →
     `default-dsc` → **YAML**.
   - Set:
     ```yaml
     spec:
       components:
         kueue:
           managementState: Removed
     ```
   - **Save**. Wait for reconciliation, then verify: `default-dsc`
     **Details** tab shows **KueueReady** `Status: False`, `Reason:
     Removed`; **Workloads** → **Deployments** (project
     `redhat-ods-applications`) no longer lists `kueue-controller-manager`.
4. Install the Red Hat build of Kueue Operator (external documentation, not
   ingested). Confirm **Status: Succeeded** in **Ecosystem** → **Installed
   Operators**.
5. Activate it in RHOAI: same `default-dsc` **YAML** tab, set
   `managementState: Unmanaged`, with either predefined or custom queue
   names:
   ```yaml
   spec:
     components:
       kueue:
         managementState: Unmanaged
   ```
   or
   ```yaml
   spec:
     components:
       kueue:
         managementState: Unmanaged
         defaultClusterQueueName: <example-cluster-queue>
         defaultLocalQueueName: <example-local-queue>
   ```
   **Save**.
6. Enable Kueue management on each existing project namespace (this is not
   automatic for pre-existing projects):
   ```
   oc label namespace <project-namespace> kueue.openshift.io/managed=true --overwrite
   ```

## Verification

- The embedded Kueue component is confirmed removed.
- `DataScienceCluster` shows a healthy `Unmanaged` status for Kueue.
- Existing queued workloads continue being processed by the Red Hat build
  of Kueue controllers; submit a new test workload to confirm.

## Next steps

- Configure quotas — see [[configuring-quota-management-for-distributed-workloads]].
- Enable Kueue in the dashboard — see
  [[configuring-kueue-workload-management]].
- Create hardware profiles for dashboard-submitted workloads ("Working
  with hardware profiles", not yet ingested).
