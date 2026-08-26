---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1967-2145
verified: 2026-08-25
confidence: documented
---

# GPU Operator controller (`spec.operator`)

"Operator component spec" — configuration for the GPU Operator's own
controller-manager pod (not one of the operands it deploys). One of the
fields `ClusterPolicy.spec` requires to be present. Unlike most other blocks
on [[gpu-operator-clusterpolicy]], this one has no `enabled`/`image`/
`repository`/`version` fields of its own.

## Fields

- `runtimeClass` (default `nvidia`) — the Kubernetes `RuntimeClass` name pods
  use to get GPU access via the container runtime the toolkit configures.
- `use_ocp_driver_toolkit` (bool) — use OpenShift's DriverToolkit image to
  build and install driver kernel modules. OpenShift-specific.
- `metrics.serviceMonitor` — Prometheus Operator `ServiceMonitor` for the GPU
  Operator's own metrics: `enabled`, `additionalLabels`, `honorLabels`,
  `interval` / `scrapeTimeout`, `relabelings` — same shape as
  [[nvidia-dcgm-exporter]]'s `serviceMonitor`.
- `annotations`, `labels` — for the operator's own resources.
- `defaultRuntime` — **deprecated**: "no longer used by the gpu-operator.
  This is instead detected at runtime."
- `initContainer` — **deprecated**: "InitContainerSpec describes
  configuration for initContainer image used with all components."
