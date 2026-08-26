---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1887-1966
verified: 2026-08-25
confidence: documented
---

# Node Status Exporter (`spec.nodeStatusExporter`)

"NodeStatusExporter spec" — one of the fields `ClusterPolicy.spec` requires to
be present. The CRD source gives no further description of what it reports
beyond the field name; do not over-claim what it exports pending a source
that says more (e.g. the GPU Operator's own docs, not yet ingested — see
`spec.toolkit`'s note and the general gap noted historically on
[[gpu-operator-clusterpolicy]]).

No fields beyond the common shape documented on [[gpu-operator-clusterpolicy]]
(`args`, `enabled`, `env`, `hostNetwork`, `image`, `imagePullPolicy`,
`imagePullSecrets`, `repository`, `resources`, `version`).
