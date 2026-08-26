---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L830-931
verified: 2026-08-24
confidence: documented
---

# NVIDIA Device Plugin (`spec.devicePlugin`)

The Kubernetes device plugin that advertises GPUs to the kubelet so pods can
request them as resources. Depends on [[nvidia-driver]]. Shares a config
surface with [[gpu-feature-discovery]] (see `config` below) and its behavior
under MIG is governed by `spec.mig.strategy`, documented on
[[nvidia-mig-manager]].

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `config` — `name`, the configmap shared between the device plugin and GFD,
  and `default`, the config name within that configmap to use.
- `mps.root` (default `/run/nvidia/mps`) — root path for NVIDIA MPS
  (Multi-Process Service) on the host, used for MPS-based GPU sharing.
