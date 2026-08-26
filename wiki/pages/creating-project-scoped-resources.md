---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L615-654
verified: 2026-08-25
confidence: documented
---

# Creating project-scoped resources (cluster administrator)

RHOAI users can access *global resources* in every project, but
*project-scoped resources* only within projects they can access. As a
cluster administrator, you can create project-scoped instances of three
resource types in any project: workbench images, hardware profiles, and
KServe model-serving runtimes. (A project member with access permissions
can do this for their own project too — see "Creating project-scoped
resources for your project", not ingested into this wiki.) Resource names
must be unique within a project.

## Preconditions

- Access to the OpenShift console as a cluster administrator.
- `disableProjectScoped` dashboard config option set to `false`.

## Steps

1. Log in to the OpenShift console as a cluster administrator.
2. Get YAML for the resource, typically by copying an existing one: in the
   **Administrator** perspective, **Home** → **Search** → select the
   `redhat-ods-applications` project → **Resources** → search by type:
   - Workbench images → `ImageStream`
   - Hardware profiles → `HardwareProfile`
   - Serving runtimes → `Template` objects whose `objects.kind` is
     `ServingRuntime`

   Select a resource → **YAML** tab → copy the content → **Cancel**.
3. Switch **Project** to the target project.
4. Toolbar **+** icon → **Import YAML**.
5. Paste the YAML.
6. Edit `metadata.namespace` to the target project name.
7. If needed, edit `metadata.name` for uniqueness within the project.
8. Optional — edit the display name shown in the RHOAI console:
   - Workbench images: `metadata.annotations.opendatahub.io/notebook-image-name`
   - Hardware profiles: `spec.displayName`
   - Serving runtimes: `objects.metadata.annotations.openshift.io/display-name`
9. **Create**.

## Verification

Log in as a regular user and confirm the resource is scoped to the
specified project: workbench images/hardware profiles appear during
[[creating-a-workbench|workbench creation]]; serving runtimes appear during
model deployment (see "Deploying models", not ingested).
