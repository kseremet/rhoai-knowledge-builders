---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L482-566
verified: 2026-08-25
confidence: documented
---

# NVIDIA DCGM (`spec.dcgm`)

"DCGM component spec" — deploys NVIDIA DCGM (Data Center GPU Manager) as its
own hostengine pod, separate from the driver container. Depends on
[[nvidia-driver]]. [[nvidia-dcgm-exporter]] scrapes this hostengine to produce
Prometheus metrics — read that page together with this one. `spec.dcgm` is
one of the fields `ClusterPolicy.spec` requires to be present.

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `hostPort` — **deprecated**: host port DCGM engine binds to (default `5555`
  per the schema description); no longer the intended configuration surface.

No other fields distinguish this component from the common shape (`args`,
`enabled`, `env`, `hostNetwork`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `resources`, `version`).

**Not the same schema as `GPUCluster`'s `spec.dcgm`.** [[gpu-operator-gpucluster]]
(the separate DRA-based CRD from the same operator) has its own `spec.dcgm`
whose description spells out a real behavioral difference — a standalone
DCGM hostengine is opt-in there, and dcgm-exporter falls back to an embedded
DCGM when it's off. `ClusterPolicy`'s `spec.dcgm` here carries no such note.
Don't assume the two CRDs' `dcgm` blocks behave identically just because the
field name and common shape match.
