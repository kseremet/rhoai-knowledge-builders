---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2695-2786
verified: 2026-08-25
confidence: documented
---

# NVIDIA vGPU Device Manager (`spec.vgpuDeviceManager`)

"VGPUDeviceManager spec" — configures which vGPU device profiles are
available on a node, analogous to how [[nvidia-mig-manager]] configures MIG
layouts. Depends on [[nvidia-vgpu-manager]] having made vGPU possible on the
host first.

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `config` — `default` (default config name `"default"` within the
  ConfigMap) and `name` (the ConfigMap of vGPU device configurations) — same
  shape as [[nvidia-mig-manager]]'s `config`.
