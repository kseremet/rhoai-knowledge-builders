---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1559-1689
verified: 2026-08-24
confidence: documented
---

# Kata Manager (`spec.kataManager`) — deprecated

The CRD schema marks this field itself: *"Deprecated: This field is no longer
honored by the GPU Operator. All values under this field are ignored."* Do not
document or recommend configuring this field for current GPU Operator
versions; it is retained in the CRD for backward compatibility only. Contrast
with [[kata-sandbox-device-plugin]], which carries no such deprecation notice
in the same schema.

Historically this provisioned Kata Containers `RuntimeClass` objects with
associated artifacts (kernel/guest images), via `config.runtimeClasses[]`
(each entry: `name`, `artifacts.url` (required, OCI artifact path),
`artifacts.pullSecret`, `nodeSelector`), staged under `config.artifactsDir`
(default `/opt/nvidia-gpu-operator/artifacts/runtimeclasses`). Depends on
[[nvidia-driver]] when it was active. Otherwise follows the common shape
documented on [[gpu-operator-clusterpolicy]].
