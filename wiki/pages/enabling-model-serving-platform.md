---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L334-349]
verified: 2026-08-25
confidence: documented
---

# Enabling the model serving platform

Enables the [[model-serving-platforms-overview|model serving platform]] cluster-wide, and enables
individual preinstalled/custom [[kserve-model-serving-crds|ServingRuntimes]] for use.

## Preconditions

- Logged in to the RHOAI dashboard as a user with OpenShift AI administrator privileges — see
  [[logging-in-to-openshift-ai]].
- KServe is installed (the `kserve` [[datasciencecluster]] component) — see
  [[installing-rhoai-components]].

## Steps

1. Enable the platform: in the left menu, **Settings → Cluster settings → General settings**,
   locate **Model serving platforms**, select the **Model serving platform** checkbox, click
   **Save changes**.
2. Enable a preinstalled runtime: **Settings → Model resources and operations → Serving
   runtimes**. This page lists preinstalled runtimes ([[accelerator-serving-runtimes]]) and any
   custom ones ([[adding-custom-model-serving-runtime]], [[adding-tested-and-verified-runtime]]).
   Set the desired runtime to **Enabled**.

## Verification

Not specified in source beyond: the model serving platform is now available for model
deployments (dashboard reflects the enabled state).

## Rollback

Not documented in this guide — clear the **Model serving platform** checkbox in step 1 to disable
platform-wide, or set a runtime back to disabled in step 2.

## See also

- [[model-serving-platforms-overview]]
- [[kserve-model-serving-crds]]
- [[accelerator-serving-runtimes]]
- [[enabling-nvidia-nim-model-serving-platform]] (depends on this procedure)
