---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1552-1586
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1587-1630
verified: 2026-08-25
confidence: documented
---

# Uninstalling RHOAI Self-Managed

Removes the [[rhoai-operator]] and every RHOAI component it installed and
manages. The **CLI method is recommended** — depending on OpenShift
version, the web console uninstall path may not prompt to remove all
associated components, leaving the cluster's final state unclear.

## What gets deleted

- The [[datasciencecluster]] instance and every per-component CR instance
  it created.
- The [[dscinitialization]] instance.
- The `Auth` CR created during/after install (see the reinstall-failure
  cause in [[troubleshooting-rhoai-operator-installation]] if this is
  missed).
- `FeatureTracker` CR instances.
- The `ServiceMesh` CR created by the Operator.
- The `KNativeServing` CR created by the Operator.
- The `redhat-ods-applications`, `redhat-ods-monitoring`, and
  `rhods-notebooks` namespaces (and workloads inside `rhods-notebooks`).
- `Subscription`, `ClusterServiceVersion`, and `InstallPlan` objects.
- `KfDef` object (version-1 Operator installs only).

## What might remain (by design)

- Projects created by users.
- Custom resource instances created by users.
- CRDs created by users or by the Operator.

These remain but are non-functional once uninstalled. Red Hat recommends
reviewing and cleaning them up afterward to avoid confusing states (e.g.
pipelines that can't run, notebooks/models that can't be undeployed).

## Preconditions

- Cluster administrator access.
- OpenShift CLI (`oc`) installed.
- Persistent disks/volumes backing your PVCs are backed up — they are not
  preserved by this process' documented scope.

## Steps

1. Open a new terminal.
2. Log in as a cluster administrator: `oc login <openshift_cluster_url> -u system:admin`
3. Create a ConfigMap that signals Operator deletion:
   `oc create configmap delete-self-managed-odh -n redhat-ods-operator`
4. Label it to trigger removal of `rhods-operator`:
   `oc label configmap/delete-self-managed-odh api.openshift.com/addon-managed-odh-delete=true -n redhat-ods-operator`
5. Once all Operator-associated objects are removed, delete the operator
   namespace. Wait for `redhat-ods-applications` to be gone first:
   ```
   PROJECT_NAME=redhat-ods-applications
   while oc get project $PROJECT_NAME &> /dev/null; do
     echo "The $PROJECT_NAME project still exists"
     sleep 1
   done
   echo "The $PROJECT_NAME project no longer exists"
   ```
   Then: `oc delete namespace redhat-ods-operator`

## Verification

1. `oc get subscriptions --all-namespaces | grep rhods-operator` — no
   result.
2. `oc get namespaces | grep -e redhat-ods* -e rhods*` — confirms
   `redhat-ods-applications`, `redhat-ods-monitoring`, `redhat-ods-operator`,
   and `rhods-notebooks` (only present if the workbenches component was
   installed) no longer exist.

## Known reinstall pitfall

If you reinstall the Operator afterward via the CLI and it fails with
`unable to find DSCInitialization` in the pod logs, the `Auth` CR from the
previous install was likely not deleted — see
[[troubleshooting-rhoai-operator-installation]] for the fix.
