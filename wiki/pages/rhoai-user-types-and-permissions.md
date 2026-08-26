---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L51-97
verified: 2026-08-25
confidence: documented
---

# RHOAI user types and permissions

RHOAI has two platform-wide user types, plus a project-scoped custom-role
mechanism for finer-grained control.

## Platform user types

| User type | Permissions |
|---|---|
| **Users** | MLOps engineers/data scientists — access and use individual components (workbenches, AI pipelines). |
| **Administrators** | Everything a user can do, plus: configure RHOAI settings; access and manage workbenches and pipeline applications for **any** project. |

By default, all OpenShift users have access to RHOAI, and OpenShift
`cluster-admin` users automatically get RHOAI administrator access.
Optionally, access can be restricted to specific users/groups — see
[[managing-rhoai-user-groups]].

Operations that require the OpenShift `cluster-admin` role specifically
(not just RHOAI administrator access):

- Adding/removing users from the RHOAI user and administrator groups (if
  using groups).
- Managing custom environment/storage configuration for users (Jupyter
  notebook resources, ConfigMaps, PVCs).

## Custom roles for project resources

Beyond the platform-wide types, project administrators can define
fine-grained, resource-level custom roles (e.g. who can view/edit/connect
to a specific [[rhoai-workbench]]) within a project:

- Custom roles are Kubernetes `Role` (scoped to one project namespace) or
  `ClusterRole` (reusable across projects) objects, created via the
  OpenShift web console or CLI.
- To surface a custom role in the RHOAI dashboard, apply the
  `opendatahub.io/dashboard: 'true'` label to it — labeled roles show an
  **AI role** badge in the dashboard's **Role type** column, distinguishing
  them from default OpenShift roles.

## Session management is separate from authentication

RHOAI and its components authenticate through OpenShift, but session
management is independent: logging out of OpenShift or RHOAI does **not**
end a logged-in Jupyter session already running. When a user's permissions
change, that user must log out of all current sessions for the change to
take effect.
