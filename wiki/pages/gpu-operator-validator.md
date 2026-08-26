---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2357-2572
verified: 2026-08-25
confidence: documented
---

# GPU Operator validator (`spec.validator`)

"Validator defines the spec for operator-validator daemonset" — runs
per-component health checks after each operand comes up. Depends on, and
checks the health of: [[nvidia-driver]] (`driver` sub-spec, described in the
schema as "Toolkit validator spec" — likely a copy-paste artifact in the
upstream schema, worth treating skeptically rather than as a precise
identification), [[nvidia-container-toolkit]] (`toolkit`), CUDA availability
(`cuda`), [[nvidia-device-plugin]] (`plugin`), [[nvidia-vfio-manager]]
(`vfioPCI`), and vGPU components (`vgpuDevices`, `vgpuManager`).

## Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]

- `driver`, `toolkit`, `cuda`, `plugin`, `vfioPCI`, `vgpuDevices`,
  `vgpuManager` — each just an `env` list scoped to that sub-validator's
  container; no independent enable/image/version per sub-validator, they
  share the top-level `image`/`repository`/`version`/`resources` of the
  validator daemonset as a whole.
