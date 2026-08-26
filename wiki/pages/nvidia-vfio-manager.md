---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2573-2694
verified: 2026-08-25
confidence: documented
---

# NVIDIA VFIO Manager (`spec.vfioManager`)

"VFIOManager for configuration to deploy VFIO-PCI Manager" — prepares GPUs
for PCI passthrough into virtual machines (`vfio-pci` driver binding), used
under `spec.sandboxWorkloads.defaultWorkload: vm-passthrough` (see
[[gpu-operator-clusterpolicy]]). Depends on [[nvidia-driver]] being managed
consistently — see `driverManager` below. Checked by [[gpu-operator-validator]]'s
`vfioPCI` sub-validator.

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `driverManager` — spec for a separate "NVIDIA Driver Manager"
  sub-component within this component: its own `env`, `image`,
  `imagePullPolicy`, `imagePullSecrets`, `repository`, `version`. The same
  pattern appears on [[nvidia-driver]] (`manager`) and [[nvidia-vgpu-manager]]
  (`driverManager`) — three independent copies of a driver-manager
  initContainer spec, not a shared one.
