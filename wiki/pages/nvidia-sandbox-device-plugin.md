---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2164-2243
verified: 2026-08-25
confidence: documented
---

# NVIDIA Sandbox Device Plugin (`spec.sandboxDevicePlugin`)

"SandboxDevicePlugin component spec" — device plugin used for
`spec.sandboxWorkloads` (VM passthrough / vGPU workloads via KubeVirt or
Kata, see the config block on [[gpu-operator-clusterpolicy]]). **This is a
separate field from [[kata-sandbox-device-plugin]]** (`spec.kataSandboxDevicePlugin`)
— the two are easy to conflate since both names contain "sandbox device
plugin," but they are distinct blocks in the CRD with distinct, correctly
scoped field descriptions here (unlike `kataSandboxDevicePlugin`, whose
descriptions are generic boilerplate — see that page).

No fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]
(`args`, `enabled`, `env`, `hostNetwork`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `resources`, `version`).
