---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L63-150
verified: 2026-08-24
confidence: documented
---

# NVIDIA CC Manager (`spec.ccManager`)

"CCManager component spec" — manages Confidential Computing (CC) mode on
compatible GPUs. Depends on [[nvidia-driver]].

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `defaultMode` (enum: `on` | `off` | `devtools`) — the default CC mode
  setting applied to compatible GPUs on the node.
