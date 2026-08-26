---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L342-509
verified: 2026-08-25
confidence: documented
---

# Installing and managing RHOAI components

Creates and edits a [[datasciencecluster]] (DSC) object to control which
RHOAI components the [[rhoai-operator]] installs. Follows
[[installing-the-rhoai-operator]] in the high-level install sequence.

## Preconditions

- The Operator is installed (see [[installing-the-rhoai-operator]]).
- Cluster administrator access.
- Custom namespaces, if used, already created (see
  [[configuring-custom-namespaces-for-rhoai]]).

## Steps — new install via CLI

1. Log in as a cluster administrator.
2. Create a `DataScienceCluster` CR, e.g. `rhods-operator-dsc.yaml`, with
   `spec.components.<name>.managementState` set for each component (see
   [[datasciencecluster]] for the full component list and the
   `Managed`/`Removed`/`Unmanaged` semantics). To use your own Argo
   Workflows instance, set `aipipelines.argoWorkflowsControllers.managementState:
   Removed` — see [[configuring-pipelines-with-own-argo-workflows]]. To use
   the predefined workbench namespace, set (or omit)
   `workbenches.workbenchNamespace: rhods-notebooks`; for a custom one, use
   the namespace from [[configuring-custom-namespaces-for-rhoai]].
   For distributed workloads (`kueue`/`ray`/`trainingoperator`), see
   [[installing-distributed-workloads-components]] instead of setting them
   ad hoc here.
3. `$ oc create -f rhods-operator-dsc.yaml`

## Steps — new install via web console

1. Log in as a cluster administrator.
2. Installed Operators → **Red Hat OpenShift AI** → **Data Science
   Cluster** tab → **Create DataScienceCluster**.
3. **Configure via** → **YAML view**; edit the same `spec.components`
   fields as the CLI path.
4. **Create**.

## Steps — changing installation status after initial install (web console only)

1. Installed Operators → **Red Hat OpenShift AI** → **Data Science
   Cluster** tab → `default-dsc` → **YAML** tab.
2. Edit `spec.components.<name>.managementState` as needed.
3. **Save** — RHOAI rolls out affected pods to the updated
   configuration/image.

## Verification

- **Pods**: Workloads → Pods, project `redhat-ods-applications` (or custom
  applications namespace) — at least one running pod per installed
  component.
- **DSC status**: Installed Operators → Red Hat OpenShift AI → Data Science
  Cluster → `default-dsc` → confirm `Phase: Ready` (may take a few minutes
  from `Progressing`). On the **YAML** tab, `status.installedComponents`
  shows `true` per installed component; a component rendered as
  `component-name: {}` under `spec.components` means it is not installed.
- **Dashboard**: help icon → **About** lists installed components with
  their upstream (source) component names and versions (Prerequisite:
  RHOAI installed; reachable even before DSC status is fully `Ready`).

## Next steps

- OpenStack/CRC/private-cloud without integrated DNS: manually configure
  DNS A/CNAME records once the LoadBalancer IP is available (external
  article, not ingested).
- KServe OAuth proxy sidecar sizing: see
  [[configuring-kserve-oauth-proxy-sidecar-resources]].
- Add users: see "Adding users to OpenShift AI user groups" (not yet
  ingested).
- Access the dashboard: see [[accessing-the-rhoai-dashboard]].
