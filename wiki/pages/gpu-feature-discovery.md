---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1455-1534
verified: 2026-08-24
confidence: documented
---

# GPU Feature Discovery / GFD (`spec.gfd`)

"GPUFeatureDiscovery spec" — labels nodes with the GPU capabilities present
(product, memory, MIG mode, etc.) so the scheduler and other components
([[nvidia-device-plugin]], [[nvidia-mig-manager]]) can act on them. Depends on
[[nvidia-driver]].

No fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]
(`args`, `enabled`, `env`, `hostNetwork`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `resources`, `version`) — this is one of the
simplest component blocks in the CRD.
