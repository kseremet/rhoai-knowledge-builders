---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L124-154
verified: 2026-08-25
confidence: documented
---

# Creating a project (RHOAI dashboard)

Creates a [[rhoai-project]].

## Preconditions

- Logged in to Red Hat OpenShift AI (see [[logging-in-to-openshift-ai]]).
- The user has the roles/permissions to create projects.

## Steps

1. From the OpenShift AI dashboard, select **Projects**.
2. Click **Create project**.
3. In the **Create project** dialog, set the **Name** field (unique display
   name).
4. Optional: click **Edit resource name** to change the OpenShift resource
   name (lowercase letters, numbers, hyphens; ≤30 chars; must start with a
   letter, end with a letter or number). **Cannot be changed after
   creation** — only the display name and description can be edited later.
5. Optional: set a **Description**.
6. Click **Create**.

## Verification

A project details page opens, from which connections, workbenches,
pipelines, and model deployments can all be added.
