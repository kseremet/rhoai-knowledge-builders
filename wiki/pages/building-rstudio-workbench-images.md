---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L218-259
verified: 2026-08-25
confidence: documented
---

# Building the RStudio Server workbench images

RStudio Server and CUDA - RStudio Server (both Technology Preview as of
RHOAI 3.5, and unavailable in disconnected environments) aren't pre-built —
build them locally before they show up as selectable [[rhoai-workbench]]
images. Red Hat supports managing the workbench; it does not support the
RStudio software itself (subject to rstudio.org's own licensing terms).

## Preconditions

- Cluster resources free: ≥1 CPU + 2Gi memory for `rstudio-server-rhel9`;
  ≥1.5 CPU + 8Gi memory for `cuda-rstudio-server-rhel9`.
- Logged in to the OpenShift cluster with the `cluster-admin` role.
- An active RHEL subscription.

## Steps

1. Create the Subscription Manager credentials secret — **must** be named
   `rhel-subscription-secret`, with `USERNAME`/`PASSWORD` keys in capital
   letters:
   `oc create secret generic rhel-subscription-secret --from-literal=USERNAME=<username> --from-literal=PASSWORD=<password> -n redhat-ods-applications`
2. Start the build. Lightweight: `oc start-build rstudio-server-rhel9 -n redhat-ods-applications --follow`.
   CUDA-enabled: `oc start-build cuda-rstudio-server-rhel9 -n redhat-ods-applications --follow`.
3. Confirm the build finished: `oc get builds -n redhat-ods-applications`
   (successful builds show `Complete`).
4. Make the image(s) selectable in the RHOAI UI by labeling their image
   streams: `oc label -n redhat-ods-applications imagestream rstudio-rhel9 opendatahub.io/notebook-image='true'`
   and/or `oc label -n redhat-ods-applications imagestream cuda-rstudio-rhel9 opendatahub.io/notebook-image='true'`.

## Verification

The built image(s) appear under **Applications** → **Enabled** in the RHOAI
dashboard, and in the **Notebook image** → **Image selection** dropdown when
creating a workbench (see [[creating-a-workbench]]).
