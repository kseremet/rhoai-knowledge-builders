---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L663-777
verified: 2026-08-25
confidence: documented
---

# Customizing component deployment resources

Edits CPU/memory requests and limits on the `Deployment` objects for RHOAI
components, all in project `redhat-ods-applications`.

## Deployment names per component

| Component | Deployment name(s) |
|---|---|
| KServe | `kserve-controller-manager`, `odh-model-controller` |
| Ray | `kuberay-operator` |
| Kueue | `kueue-controller-manager` |
| Workbenches | `notebook-controller-deployment`, `odh-notebook-controller-manager` |
| Dashboard | `rhods-dashboard` |
| Model serving | `modelmesh-controller`, `odh-model-controller` |
| Model registry | `model-registry-operator-controller-manager` |
| AI pipelines | `data-science-pipelines-operator-controller-manager` |
| Training Operator | `kubeflow-training-operator` |

## Customizing component resources

**Precondition**: cluster administrator privileges.

1. OpenShift console, **Administrator** perspective → **Workloads** →
   **Deployments** → project `redhat-ods-applications`.
2. Click the deployment name for the component (see table above) → **YAML**.
3. Edit `.spec.template.spec.containers.resources`, e.g. to set a 500Mi
   memory limit:
   ```yaml
   containers:
   - resources:
       limits:
         cpu: '2'
         memory: 500Mi
       requests:
         cpu: '1'
         memory: 1Gi
   ```
4. **Save**, then **Reload**.

**Verification**: log in to RHOAI and confirm the resource change applies.

**Persistence caveat**: customizations persist only while the
`opendatahub.io/managed: true` annotation is **absent** (the default) from
the deployment's YAML — see the next section.

## Disabling component resource customization

Adding `opendatahub.io/managed: true` to a component deployment restores
Operator-managed defaults and stops accepting manual resource edits.

**Important**: manually removing this annotation, or setting it to
`false`, after having added it, may cause unexpected cluster issues — to
undo it, delete the deployment instead (see next section).

**Precondition**: cluster administrator privileges.

1. **Workloads** → **Deployments** → `redhat-ods-applications` → open the
   component's deployment → **YAML**.
2. Under `metadata.annotations`, add:
   ```yaml
   metadata:
     annotations:
       opendatahub.io/managed: true
   ```
3. **Save**, then **Reload**.

**Verification**: the annotation appears in the deployment's YAML.

## Re-enabling component resource customization

To remove the `opendatahub.io/managed: true` annotation safely, delete the
deployment rather than editing the annotation directly — the controller pod
automatically redeploys it with default settings (and the annotation gone).

**Precondition**: cluster administrator privileges.

1. **Workloads** → **Deployments** → `redhat-ods-applications` → open the
   component's deployment.
2. Options menu → **Delete Deployment**.

**Verification**: the controller pod redeploys the deployment automatically
with default settings.
