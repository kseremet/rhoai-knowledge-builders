---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L591-619
verified: 2026-08-25
confidence: documented
---

# Accessing the RHOAI dashboard (admin: retrieving and sharing the URL)

The cluster-administrator-side counterpart to [[logging-in-to-openshift-ai]]:
retrieves the dashboard URL from the OpenShift web console so it can be
shared with data scientists after install and after users are added to the
RHOAI user group.

## Preconditions

- RHOAI is installed (see [[installing-rhoai-components]]).
- At least one user has been added to the RHOAI user group (see "Adding
  users to OpenShift AI user groups", not yet ingested).

## Steps

1. Log in to the OpenShift web console.
2. Click the application launcher.
3. Right-click **Red Hat OpenShift AI** and copy the URL.
4. Share this URL with data scientists so they can log in (see
   [[logging-in-to-openshift-ai]]).

**Important — as of 3.4 EA1**: the dashboard URL format is
`https://rh-ai.apps.<cluster-domain>`. Upgrades from earlier versions
auto-redirect old bookmarks (including legacy `rhods-dashboard` or
`data-science-gateway` URLs) to the new `rh-ai` format — but you must still
give users and administrators the new URL directly.

## Verification

Confirm you and your users can log in using the instance URL.

## See also

Viewing installed components from the dashboard's **About** page is covered
on [[logging-in-to-openshift-ai]].
