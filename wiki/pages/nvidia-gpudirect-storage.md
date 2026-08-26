---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1408-1454
verified: 2026-08-24
confidence: documented
---

# NVIDIA GPUDirect Storage / GDS (`spec.gds`)

The upstream schema itself labels this **"Experimental"** ("GPUDirectStorage
defines the spec for GDS components (Experimental)") — treat it as such rather
than a supported production path. Depends on [[nvidia-driver]]; lets storage
I/O move directly to/from GPU memory, bypassing a CPU bounce buffer.

Fields: `args`, `enabled`, `env`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `version` — same reduced shape as
[[nvidia-gdrcopy]] (no `hostNetwork`, no `resources`).
