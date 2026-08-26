---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2787-2926
verified: 2026-08-25
confidence: documented
---

# NVIDIA vGPU Manager (`spec.vgpuManager`)

"VGPUManager component spec" — installs the NVIDIA vGPU host driver so the
node can carve GPUs into virtual GPU instances for VMs, used under
`spec.sandboxWorkloads.defaultWorkload: vm-vgpu` (see
[[gpu-operator-clusterpolicy]]). Works with [[nvidia-vgpu-device-manager]]
(which configures the vGPU *profiles*, once the host manager here has made
vGPU possible) and is checked by [[gpu-operator-validator]]'s `vgpuManager`
sub-validator.

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `kernelModuleConfig.name` — configmap with kernel module config parameters
  for the vGPU manager, mirroring [[nvidia-driver]]'s `kernelModuleConfig`.
- `driverManager` — a separate "NVIDIA Driver Manager" initContainer spec
  (`env`, `image`, `imagePullPolicy`, `imagePullSecrets`, `repository`,
  `version`) — see the note on [[nvidia-vfio-manager]] about the three
  independent copies of this pattern in the CRD.
