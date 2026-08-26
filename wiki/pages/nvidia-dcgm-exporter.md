---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L567-829
verified: 2026-08-24
confidence: documented
---

# NVIDIA DCGM Exporter (`spec.dcgmExporter`)

Exposes GPU metrics as Prometheus samples, scraping [[nvidia-dcgm]]'s
hostengine. Depends on [[nvidia-dcgm]] and, transitively, [[nvidia-driver]].

Fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]:

- `config.name` — configmap holding a `dcgm-metrics.csv` file that customizes
  which metrics are collected.
- `enablePodLabels` / `enablePodUID` (bool) — add Kubernetes pod labels / pod
  UID as Prometheus label dimensions on exported metrics. Both require
  cluster-level read access to pods.
- `podLabelAllowlistRegex` — list of regexes filtering which pod labels are
  included when `enablePodLabels` is set; empty means all labels are included.
- `hostPID` (bool) — access the host's PID namespace.
- `hpcJobMapping` — `enabled` (bool) plus `directory` (default
  `/var/lib/dcgm-exporter/job-mapping`) for HPC job-mapping files written by
  the workload manager.
- `service` — `type` (Kubernetes ServiceType) and `internalTrafficPolicy`.
- `serviceMonitor` — Prometheus Operator `ServiceMonitor` config: `enabled`,
  `additionalLabels`, `honorLabels`, `interval` / `scrapeTimeout` (Prometheus
  duration strings, e.g. `30s`), and `relabelings` (the standard Prometheus
  `RelabelConfig` list: `action`, `sourceLabels`, `separator`, `regex`,
  `modulus`, `targetLabel`, `replacement`).
- `annotations` — annotations for the exporter's own resources (separate from
  the `daemonsets`-wide annotations on [[gpu-operator-clusterpolicy]]).
