---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L79-123
verified: 2026-08-25
confidence: documented
---

# Logging in to OpenShift AI

## Preconditions

- RHOAI is installed on the OpenShift cluster — see
  [[installing-the-rhoai-operator]] and [[installing-rhoai-components]]
  (disconnected-environment install guide not yet ingested).
- The RHOAI identity provider and login credentials are known — a data
  scientist/data engineer/ML engineer gets the instance URL (e.g.
  `rh-ai.apps.example.abc.p1.openshiftapps.com`) from their administrator.
- A current Chrome, Firefox, or Safari.

## Steps

1. Browse to the OpenShift AI instance URL and click **Log in with
   OpenShift** — or, from the OpenShift web console, **Application
   Launcher** → **Red Hat OpenShift AI**.
2. Click the identity provider (GitHub, Google, company SSO, ...).
3. Enter credentials, click **Log in**.

## Verification

The OpenShift AI dashboard opens on the **Home** page.

## Related: viewing installed components

From the dashboard, click the help icon (top nav) → **About** to see the
installed RHOAI components alongside their upstream (source) component names
and versions.
