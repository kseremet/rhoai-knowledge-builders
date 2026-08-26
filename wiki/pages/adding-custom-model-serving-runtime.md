---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L379-409]
verified: 2026-08-25
confidence: documented
---

# Adding a custom model-serving runtime

Adds and enables a custom [[kserve-model-serving-crds|ServingRuntime]] via the RHOAI dashboard,
for cases where preinstalled runtimes ([[accelerator-serving-runtimes]]) don't meet your needs.
Red Hat does not support custom runtimes — you are responsible for licensing, configuring, and
maintaining them.

## Preconditions

- Logged in to the RHOAI dashboard as an OpenShift AI administrator.
- The custom runtime is built and its image is pushed to a container image repository (e.g.
  Quay).

## Steps

1. **Settings → Model resources and operations → Serving runtimes** — shows already-installed/
   enabled runtimes.
2. Either duplicate an existing runtime (action menu ⋮ → **Duplicate**, e.g. starting from **vLLM
   NVIDIA GPU ServingRuntime for KServe**) or start fresh (**Add serving runtime**).
3. In **Select the model serving platforms this runtime supports**, select **Single-model serving
   platform**.
4. In **Select the API protocol this runtime supports**, select **REST** or **gRPC**.
5. If starting fresh, supply the YAML: **Upload files** (choose a local YAML file) or **Start
   from scratch** and paste YAML directly. Custom runtimes commonly need custom parameters added
   to the ServingRuntime spec's `env` section.
6. Click **Add**. The runtime is added and auto-enabled; the chosen API protocol is shown.
7. Optional: edit later via action menu (⋮) → **Edit**.

## Verification

The custom runtime appears in an enabled state on the **Serving runtimes** page.

## Rollback

Not documented — disable or remove the runtime via its action menu (⋮) on the **Serving
runtimes** page.

## See also

- [[kserve-model-serving-crds]]
- [[accelerator-serving-runtimes]]
- [[adding-tested-and-verified-runtime]]
- [[enabling-model-serving-platform]]
