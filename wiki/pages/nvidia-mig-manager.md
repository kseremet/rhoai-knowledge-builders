---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1770-1781
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1782-1886
verified: 2026-08-24
confidence: documented
---

# NVIDIA MIG Manager (`spec.mig`, `spec.migManager`)

MIG (Multi-Instance GPU) partitioning is split across two `spec` blocks:

- `spec.mig.strategy` (enum: `none` | `single` | `mixed`) — the MIG strategy
  applied by both [[gpu-feature-discovery]] and [[nvidia-device-plugin]], not
  just by the MIG Manager itself. This is a cluster-wide switch, not a
  per-component one.
- `spec.migManager` — the MIG Manager component that actually reconfigures
  GPUs into MIG instances. Depends on [[nvidia-driver]]; coordinates with
  [[gpu-feature-discovery]] and [[nvidia-device-plugin]] since a MIG layout
  change changes what those two report/advertise.

## `migManager` fields beyond the common shape

(documented on [[gpu-operator-clusterpolicy]]: `args`, `enabled`, `env`,
`hostNetwork`, `image`, `imagePullPolicy`, `imagePullSecrets`, `repository`,
`resources`, `version`)

- `config` — `default` (enum: `all-disabled` | `""`, default `all-disabled`),
  the MIG config applied to a node when it has no `nvidia.com/mig.config`
  node label; `name`, a configmap of mig-parted configs — if unset, MIG
  configuration is generated dynamically from the hardware instead.
- `gpuClientsConfig.name` — configmap with a custom gpu-clients list for the
  MIG Manager container.
