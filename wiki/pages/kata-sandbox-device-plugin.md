---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1690-1769
verified: 2026-08-24
confidence: documented
---

# Kata Sandbox Device Plugin (`spec.kataSandboxDevicePlugin`)

A device plugin variant for GPU access inside Kata Containers sandboxes.
Unlike [[kata-manager]], the CRD does not mark this field deprecated — it
appears to remain live, though the schema's `description` fields for this
component are generic ("NVIDIA component image name" / "NVIDIA component
image repository" / "NVIDIA component image tag") rather than naming the
plugin specifically, so treat those descriptions as boilerplate rather than a
precise identification. Depends on [[nvidia-driver]]; related to
[[kata-manager]] for the Kata runtime-class side of the setup.

No fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]
(`args`, `enabled`, `env`, `hostNetwork`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `resources`, `version`).
