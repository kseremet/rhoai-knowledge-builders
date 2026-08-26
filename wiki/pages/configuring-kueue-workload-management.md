---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1053-1121
verified: 2026-08-25
confidence: documented
---

# Configuring workload management with Kueue

Installs the Red Hat build of Kueue Operator and activates the
[[kueue-workload-management|Kueue integration]] in RHOAI, then optionally
surfaces it in the dashboard.

## Preconditions

- Cluster administrator privileges.
- OpenShift 4.19 or later.
- **cert-manager Operator for Red Hat OpenShift** installed and configured.
- OpenShift CLI (`oc`) installed.

## Steps — activating the integration

1. `oc login <openshift_cluster_url> -u <admin_username> -p <password>`
2. Install the Red Hat build of Kueue Operator (external documentation, not
   ingested).
3. Activate the integration on the [[datasciencecluster]] CR. Using the
   predefined `default` queue names:
   ```
   oc patch datasciencecluster default-dsc --type='merge' -p \
     '{"spec":{"components":{"kueue":{"managementState":"Unmanaged"}}}}' \
     -n <operator-namespace>
   ```
   Or with custom queue names:
   ```
   oc patch datasciencecluster default-dsc --type='merge' -p \
     '{"spec":{"components":{"kueue":{"managementState":"Unmanaged","defaultClusterQueueName":"<example-cluster-queue>","defaultLocalQueueName":"<example-local-queue>"}}}}' \
     -n <operator-namespace>
   ```
   `<operator-namespace>` defaults to `redhat-ods-operator`.

## Verification

- `oc get pods -n openshift-kueue-operator` shows `kueue-controller-manager-*`
  and `openshift-kueue-operator-*` pods `Running`.
- `oc get clusterqueues` shows the default `ClusterQueue`.

## Enabling Kueue in the dashboard

Optional next step. Once enabled, RHOAI auto-labels **new** dashboard-created
project namespaces `kueue.openshift.io/managed=true` and creates a
`LocalQueue` (annotated `opendatahub.io/managed=false`, so it's freely
editable/deletable afterward) if one doesn't exist. A validating webhook
then enforces the `kueue.x-k8s.io/queue-name` label on new/updated
workload resources in that namespace.

**Existing** projects, or ones created via `oc`, are **not** auto-labeled —
apply it manually:
```
oc label namespace <project-namespace> kueue.openshift.io/managed=true --overwrite
```

**Preconditions**: cluster administrator privileges; OpenShift 4.19+; the
Red Hat build of Kueue Operator installed and activated (above); quotas
already configured.

1. `oc login <openshift_cluster_url> -u <admin_username> -p <password>`
2. Patch the dashboard config in the applications namespace (default
   `redhat-ods-applications`):
   ```
   oc patch odhdashboardconfig odh-dashboard-config \
     -n <applications-namespace> \
     --type merge \
     -p {"spec":{"dashboardConfig":{"disableKueue":false}}}
   ```

**Verification**:

1. Create a new project from the dashboard.
2. Confirm the namespace is labeled:
   ```
   oc get ns <project-namespace> -o jsonpath='{.metadata.labels.kueue\.openshift\.io/managed}{"\n"}'
   ```
   Expect `true`.
3. Confirm a default `LocalQueue` exists:
   `oc get localqueues -n <project-namespace>`
4. Create a test workload (e.g. a `Notebook`) and confirm it carries the
   `kueue.x-k8s.io/queue-name` label.

## Next steps

- Configure quotas — see [[configuring-quota-management-for-distributed-workloads]].
- Create hardware profiles so users can submit workloads from the
  dashboard ("Working with hardware profiles", not yet ingested).
