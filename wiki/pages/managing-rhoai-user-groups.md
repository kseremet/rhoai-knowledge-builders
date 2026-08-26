---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L98-185
verified: 2026-08-25
confidence: documented
---

# Managing RHOAI user and administrator groups

Covers viewing, adding users to, and selecting the OpenShift groups that
RHOAI uses to restrict access — see [[rhoai-user-types-and-permissions]]
for what the **user group** vs **administrator group** each grant. By
default all OpenShift users can access RHOAI; these groups are only
consulted if you choose to restrict access.

Group membership itself is always managed in the OpenShift web console, not
the RHOAI dashboard — RHOAI only lets you *select which* OpenShift groups
count as its user/administrator groups.

## Preconditions

- A supported OpenShift identity provider is configured.
- The `cluster-admin` role (for viewing/adding users to groups).
- If using LDAP: LDAP group syncing to OpenShift must be configured
  separately (not covered here).

## Viewing OpenShift AI users

1. OpenShift web console → **User Management** → **Groups**.
2. Click the group name (e.g. `rhods-admins` for administrators,
   `rhods-users` for regular users).
3. The **Users** section of **Group details** lists the members.

## Adding users to OpenShift AI user groups

1. OpenShift web console → **User Management** → **Groups**.
2. Click the group to add users to (`rhods-admins` or `rhods-users`).
3. **Actions** → **Add Users**.
4. Enter the user name in the **Users** field.
5. **Save**.

**Verification**: the **Details** tab for the group lists the added user
names.

## Selecting OpenShift AI administrator and user groups

Once a `cluster-admin` has defined the groups in OpenShift, an RHOAI
administrator selects them for RHOAI's own use:

1. RHOAI dashboard → **Settings** → **User management**.
2. Under **Red Hat OpenShift AI administrator groups**, select an OpenShift
   group (repeat to add more).
3. Under **Red Hat OpenShift AI user groups**, select an OpenShift group
   (repeat to add more). **Important**: the special value
   `system:authenticated` allows *all* authenticated OpenShift users into
   RHOAI.
4. **Save changes**.

**Verification**: administrator users can log in and see the **Settings**
menu; non-administrator users can log in and use projects/workbenches.
Users not in either selected group cannot view the dashboard, use
associated services (e.g. Jupyter), or access **Cluster settings**.

## See also

[[deleting-rhoai-users]] — the reverse operation (removing a user from
these same groups as part of a full user teardown).
