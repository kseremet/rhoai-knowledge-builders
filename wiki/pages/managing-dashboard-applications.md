---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L472-614
verified: 2026-08-25
confidence: documented
---

# Managing applications shown in the RHOAI dashboard

Five related admin tasks that control the **Applications** → **Enabled** /
**Explore** pages of the RHOAI dashboard: adding a tile for an
already-installed application, disabling new tile-adding, disabling an
application entirely, showing/hiding tile metadata, and hiding the built-in
basic-workbench tile. All but the second (Operator-uninstall based) edit
custom resources via **Home** → **API Explorer** in the OpenShift console's
**Administrator** perspective, in project `redhat-ods-applications`.

## Adding an application to the dashboard

Adds a tile for an application already installed in the OpenShift cluster.

**Preconditions**: RHOAI administrator privileges;
`spec.dashboardConfig.enablement` is `true` (default).

1. API Explorer → search `OdhApplication` → open the CR → project
   `redhat-ods-applications` → **Instances** → **Create OdhApplication**.
2. Paste and edit an `OdhApplication` YAML manifest, e.g.:
   ```yaml
   apiVersion: dashboard.opendatahub.io/v1
   kind: OdhApplication
   metadata:
     name: examplename
     namespace: redhat-ods-applications
     labels:
       app: odh-dashboard
       app.kubernetes.io/part-of: odh-dashboard
   spec:
     enable:
       validationConfigMap: examplename-enable
     img: >-
       <svg ...>...</svg>
     getStartedLink: 'https://example.org/docs/quickstart.html'
     route: exampleroutename
     routeNamespace: examplenamespace
     displayName: Example Name
     kfdefApplications: []
     support: third party support
     csvName: ''
     provider: example
     docsLink: 'https://example.org/docs/index.html'
     quickStart: ''
     getStartedMarkDown: >-
       # Example
       Enter text for the information panel.
     description: >-
       Enter summary text for the tile.
     category: Self-managed | Partner managed | Red Hat managed
   ```
   (View an existing instance's YAML under the same **Instances** tab for a
   worked example.)
3. **Create**.
4. In the RHOAI dashboard: **Applications** → **Explore** → find the new
   tile → click it → **Enable** in the info panel.

**Verification**: **Applications** → **Enabled** shows the application.

## Preventing users from adding applications to the dashboard

Disables the "add application to dashboard" ability for RHOAI
administrators cluster-wide.

**Precondition**: RHOAI administrator privileges. Note: this does not hide
the **Start basic workbench** tile — see the dedicated section below for
that.

1. API Explorer → search `OdhDashboardConfig` → open the `odh-dashboard-config`
   instance in `redhat-ods-applications` → **YAML**.
2. Under `spec.dashboardConfig`, set `enablement: false`.
3. **Save**, then **Reload** to sync the change.

**Verification**: check the **Applications** → **Enabled** page.

## Disabling applications connected to OpenShift AI

Uninstalls the application/component's underlying Operator so its tile no
longer shows.

**Preconditions**: OpenShift web console access; `cluster-admins` group
membership; the application/component is currently installed and visible
on **Enabled**.

1. Switch to the **Administrator** perspective, project
   `redhat-ods-applications`.
2. Installed Operators page (**Ecosystem** → **Installed Operators** on
   OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19).
3. Click the Operator to remove; delete any Operator-managed resources or
   instances first (required before uninstall can complete).
4. **Actions** → **Uninstall Operator** → confirm.

**Important**: uninstalling the Operator does **not** remove its CRDs or
already-managed resources, or clean up any off-cluster resources it
configured — those need manual cleanup.

**Verification**: the Operator is gone from **Installed Operators**; the
application shows **Disabled** on the **Enabled** page (may take a few
minutes after Operator removal).

## Showing or hiding information about available applications

Controls two pieces of per-tile metadata on the **Exploring applications**
page: the support-level badge (`Red Hat-managed` / `Partner managed` /
`Self-managed`) and the info panel shown when a tile is clicked.

**Precondition**: RHOAI administrator privileges.

1. API Explorer → `OdhDashboardConfig` → `odh-dashboard-config` instance in
   `redhat-ods-applications` → **YAML**.
2. Under `spec.dashboardConfig`, set either:
   - `disableInfo: true` to hide the info panel (`false` = shown, default).
   - `disableISVBadges: true` to hide the support-level badge (`false` =
     shown, default).
3. **Save**, then **Reload**.

**Verification**: log in to the dashboard and confirm the configured
behavior.

## Hiding the default basic workbench application

Removes the **Start basic workbench** tile, which is enabled by default and
unaffected by the "preventing users from adding applications" toggle above.

**Precondition**: RHOAI administrator privileges.

1. API Explorer → `OdhDashboardConfig` → `odh-dashboard-config` instance in
   `redhat-ods-applications` → **YAML**.
2. Under `spec:notebookController`, set `enabled: false`.
3. **Save**, then **Reload**.

**Verification**: **Applications** → **Enabled** no longer lists **Start
basic workbench**.
