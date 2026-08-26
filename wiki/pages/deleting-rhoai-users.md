---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L186-287
verified: 2026-08-25
confidence: documented
---

# Deleting a user from RHOAI

Full teardown of a user's RHOAI access and resources. Four tasks, run in
order: stop the user's basic workbenches, revoke their access (remove from
[[managing-rhoai-user-groups|the RHOAI groups]]), back up their data, then
clean up their leftover cluster objects.

Before deleting a user, back up the data on their PVCs — see
[[backing-up-rhoai-data]].

## 1. Stopping basic workbenches owned by other users

**Preconditions**: logged in with RHOAI administrator privileges; the
**Start basic workbench** application is running for the target user.

1. On the **Start basic workbench** page, click the **Administration** tab.
2. To stop one user's workbench: locate them in the **Users** section, then
   either click the action menu (⋮) → **Stop server**, or click **View
   server** → **Stop workbench** → confirm **Stop server**.
3. To stop every workbench at once: click **Stop all workbenches** → **OK**.

**Verification**: the **Stop server** link beside each server changes to
**Start workbench**.

## 2. Revoking user access to basic workbenches

Only applicable if you've restricted access via RHOAI user groups (see
[[managing-rhoai-user-groups]]). Removing a user from those groups blocks
dashboard access and associated services (e.g. Jupyter) but does **not** by
itself remove them from your identity provider.

**Preconditions**: the user's workbenches are already stopped;
`cluster-admin` role; the user is currently a member of the user group,
administrator group, or both.

1. OpenShift web console → **User Management** → **Groups**.
2. Click the relevant group (`rhods-admins` or `rhods-users`).
3. On the **Details** tab, locate the user in the **Users** section.
4. Action menu (⋮) beside the user → **Remove user**.

**Verification**: the user no longer appears in the group's **Users**
section. In **Workloads** → **Pods** (project `rhods-notebooks` or your
custom workbench namespace), confirm there is no `jupyter-nb-<username>-*`
pod for them — if one exists, delete it manually. In the RHOAI dashboard,
delete any projects belonging to the user.

To completely remove the user (not just from RHOAI), also remove them from
the allowed group in your identity provider.

## 3. Backing up storage data

Best practice before deleting a user (all their PVCs are removed). See
[[backing-up-rhoai-data]] for the full procedure.

## 4. Cleaning up after deleting users

Removes the user's leftover per-user cluster objects. Do this after backing
up their data.

**Preconditions**: user's storage data backed up; optionally, their
credentials removed from the identity provider; logged in as
`cluster-admin`.

1. Delete the user's PVC: **Storage** → **PersistentVolumeClaims** →
   project `rhods-notebooks` (or custom workbench namespace) → locate
   `jupyter-nb-<username>` → action menu (⋮) → **Delete
   PersistentVolumeClaim** → confirm → **Delete**.
2. Delete the user's ConfigMap: **Workloads** → **ConfigMaps** → same
   project → locate `jupyterhub-singleuser-profile-<username>` → action
   menu (⋮) → **Delete ConfigMap** → confirm → **Delete**.

**Verification**: the user sees an "Access permission needed" message if
they try to access RHOAI; their single-user profile, PVC, and ConfigMap are
gone from OpenShift.
