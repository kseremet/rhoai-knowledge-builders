---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2273-2356
verified: 2026-08-25
confidence: documented
---

# NVIDIA Container Toolkit (`spec.toolkit`)

Exposes the GPU driver to the container runtime — this is what most other
[[gpu-operator-clusterpolicy]] components actually need at runtime, since
without it a container runtime cannot see the GPU devices [[nvidia-driver]]
loaded. `spec.toolkit` is one of the fields `ClusterPolicy.spec` requires to
be present.

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `installDir` (default `/usr/local/nvidia`) — where the toolkit is installed
  on the host.

[[gpu-operator-validator]]'s `toolkit` sub-check validates this component
came up correctly.
