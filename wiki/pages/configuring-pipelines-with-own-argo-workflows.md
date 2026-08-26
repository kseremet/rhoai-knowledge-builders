---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L510-541
verified: 2026-08-25
confidence: documented
---

# Configuring pipelines with your own Argo Workflows instance

Disables the embedded Argo Workflows controller that ships with the
`aipipelines` component of [[datasciencecluster]], so AI pipelines can use
an existing, independently-managed Argo Workflows instance on the cluster
instead. Useful when the cluster already runs a managed Argo Workflows
install and you want to avoid two controllers conflicting, and to manage
the lifecycles of RHOAI and Argo Workflows independently.

**You cannot run both** the embedded and your own Argo Workflows instance
on the same cluster at once.

## Preconditions

- Cluster administrator access.
- RHOAI is installed (see [[installing-rhoai-components]]).

## Steps

1. Log in to the web console as a cluster administrator.
2. Installed Operators → **Red Hat OpenShift AI** → **Data Science
   Cluster** tab → `default-dsc` → **YAML** tab.
3. Set `spec.components.aipipelines.managementState: Managed` and, under
   it, `argoWorkflowsControllers.managementState: Removed`:
   ```yaml
   spec:
     components:
       aipipelines:
         argoWorkflowsControllers:
           managementState: Removed
         managementState: Managed
   ```
4. **Save**.
5. Install and configure a compatible Argo Workflows version on the cluster
   yourself (see [Supported Configurations for 3.x](https://access.redhat.com/articles/rhoai-supported-configs-3.x)
   and the [Argo Workflows installation docs](https://argo-workflows.readthedocs.io/en/stable/installation/)
   — external, not ingested).

## Verification

1. `DataScienceCluster` **Details** tab → `AIPipelinesReady` condition
   **Status** is `True`.
2. Confirm the `ds-pipeline-workflow-controller` pod does **not** exist
   (Workloads → Pods, search by name) — its absence confirms the embedded
   controller is disabled.
