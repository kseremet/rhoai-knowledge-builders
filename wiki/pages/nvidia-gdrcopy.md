---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1361-1407
verified: 2026-08-24
confidence: documented
---

# NVIDIA GDRCopy (`spec.gdrcopy`)

Deploys the GDRCopy driver, which lets code copy data directly between GPU
memory and CPU memory without staging through host buffers. Depends on
[[nvidia-driver]].

Fields: `args`, `enabled`, `env`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `version` — the common shape from
[[gpu-operator-clusterpolicy]] minus `hostNetwork` and `resources`, which this
component's schema does not define (unlike most other components).
