---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L447-483]
verified: 2026-08-25
confidence: documented
---

# Enabling the NVIDIA NIM model serving platform

Enables the [[model-serving-platforms-overview|NVIDIA NIM model serving platform]] via the RHOAI
dashboard, so NVIDIA-optimized models can be deployed as NIMs (a specific NIM, its replica count,
server size, and hardware profile are chosen during the deployment itself — not covered by this
procedure).

## Preconditions

- Logged in to the RHOAI dashboard as an OpenShift AI administrator.
- The [[enabling-model-serving-platform|model serving platform]] is already enabled (no need to
  also enable a preinstalled runtime for this).
- The dashboard's `disableNIMModelServing` configuration option is `false`.
- GPU support is enabled in OpenShift AI: Node Feature Discovery Operator and NVIDIA GPU Operator
  installed — see [[enabling-nvidia-gpus]].
- An NVIDIA Cloud Account (NCA) with access to the NVIDIA GPU Cloud (NGC) portal, associated with
  the NVIDIA AI Enterprise Viewer role.
- A personal API key generated on the NGC portal.

If you previously enabled this platform and then upgraded RHOAI, re-enter your NVIDIA personal
API key to re-enable it.

## Steps

1. **Applications → Explore** in the left menu.
2. Find the **NVIDIA NIM** tile.
3. Click **Enable** on the tile.
4. Enter your personal API key, click **Submit**.

## Verification

The NVIDIA NIM application appears on the **Enabled** page.

## Rollback

Not documented in this guide.

## See also

- [[model-serving-platforms-overview]]
- [[enabling-model-serving-platform]]
- [[enabling-nvidia-gpus]]
