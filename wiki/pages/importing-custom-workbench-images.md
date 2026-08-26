---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L421-471
verified: 2026-08-25
confidence: documented
---

# Enabling and importing custom workbench images

Makes a [[creating-custom-workbench-images|built custom workbench image]]
selectable when a data scientist creates a [[rhoai-workbench]].

## Enabling the Workbench images menu

By default, RHOAI administrators can already import custom images via
**Settings** → **Environment setup** → **Workbench images**. If that path
isn't available:

- **`Settings` menu missing entirely**: menu visibility follows RHOAI
  admin permissions (members of `rhods-admins`; OpenShift `cluster-admin`
  users are auto-added to that group). See
  [[rhoai-user-types-and-permissions]].
- **`Workbench images` missing under `Settings`**: controlled by the
  `dashboardConfig: disableBYONImageStream` field on `OdhDashboardConfig`
  (default `false` = visible). Requires RHOAI administrator permissions to
  edit.

## Importing a custom workbench image

**Preconditions**: RHOAI administrator privileges; the custom image exists
in a registry reachable from RHOAI; the **Workbench images** menu item is
enabled (above); if associating an accelerator — its identifier string,
plus GPU support already enabled in RHOAI (NFD Operator + accelerator
Operator; see [[enabling-nvidia-gpus]] for the NVIDIA case).

1. Dashboard → **Settings** → **Environment setup** → **Workbench images**.
   Previously-imported images are listed; toggle the **Enable** column to
   turn one on/off.
2. Optional: to associate an accelerator without an existing accelerator or
   hardware profile, click **Create profile** on the image's row (if the
   image lacks an accelerator identifier, configure one manually first).
3. Click **Import new image** (or **Import image** if none exist yet).
4. **Image location**: registry URL, e.g. `quay.io/my-repo/my-image:tag`,
   `quay.io/my-repo/my-image@sha256:...`, or `docker.io/my-repo/my-image:tag`.
5. **Name**, and optionally **Description**.
6. Optional: **Accelerator identifier** — pick one from the list (defaults
   to the image's identifier if it has only one).
7. Optional: **Software** tab → **Add software** → edit name/version →
   confirm, for each software entry to display on the workbench-creation
   page.
8. Optional: **Packages** tab → **Add package** → edit name/version (e.g.
   `3.16.7`) → confirm, for each package entry to display.
9. Click **Import**.

**Verification**: the image appears in the **Workbench images** table, and
is selectable when a user creates a [[rhoai-workbench|workbench]].
