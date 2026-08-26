---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml.md#L21-31
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml.md#L64-68
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml.md#L69-378
verified: 2026-08-25
confidence: documented
---

# GPU Operator GPUCluster (DRA-based, alpha)

`GPUCluster` (group `nvidia.com`, kind `GPUCluster`, short name `gc`, scope
`Cluster`, version `v1alpha1`) is a second custom resource from the same
[NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator) repo as
[[gpu-operator-clusterpolicy]] — a distinct, newer, DRA-based (Dynamic
Resource Allocation) software-enablement path, not a version or successor of
`ClusterPolicy`. Its schema description states explicitly: *"Unlike
ClusterPolicy, it does not manage the NVIDIA driver or the device-plugin;
the driver is installed separately (host-installed or via an NVIDIADriver
CR) and GPUCluster waits for driver readiness before proceeding."* Same
short-output columns as `ClusterPolicy`: `Status` from `.status.state`, `Age`
from `.metadata.creationTimestamp`.

Being `v1alpha1`, treat this CRD as unstable/experimental relative to
`ClusterPolicy`'s `v1`.

## Ingest status: partial

Only section 1 of 3 of this source document has been ingested so far. What's
confirmed below is real; what's *not* mentioned (most of `spec.dcgm`,
`spec.dcgmExporter`, `spec.draDriver`, `spec.hostPaths`, and the full
`status` schema) simply hasn't been read yet — don't infer it's absent from
the CRD.

## What's confirmed so far

**`spec.daemonsets`** (`#L69-378`) — identical schema to
[[gpu-operator-clusterpolicy]]'s `spec.daemonsets`: `annotations`, `labels`,
`podSecurityContext`, `priorityClassName`, `tolerations`,
`rollingUpdate.maxUnavailable`, `updateStrategy`.

**`spec.dcgm`** — partially read. Its description differs meaningfully from
`ClusterPolicy`'s `spec.dcgm` and is worth quoting: *"DCGM defines the spec
for the standalone NVIDIA DCGM hostengine. Disabled by default; when
disabled, dcgm-exporter uses its embedded nv-hostengine. NOTE: the reused
enabled field carries no server-side default and its IsEnabled() treats nil
as enabled, so the controller must default nil enabled to disabled here."*
In other words: on `GPUCluster`, a standalone [[nvidia-dcgm]] hostengine is
opt-in, and [[nvidia-dcgm-exporter]] falls back to an embedded DCGM instead
of talking to a separate pod when it's off — a real behavioral difference
from `ClusterPolicy`, where `spec.dcgm` carries no such note. Fields
confirmed present: `args`, `enabled`, `env`, `hostNetwork`, `hostPort`
(deprecated, same as `ClusterPolicy`'s), `image`. Cut off before
`imagePullPolicy`'s description and everything after — do not assume the
rest of the common shape is absent, it just wasn't in this ingest unit.

**`spec.draDriver`** — field exists (seen in the field list) but no content
ingested yet. This is presumably the DRA (Dynamic Resource Allocation) driver
component that makes `GPUCluster` different from `ClusterPolicy` in the first
place — a priority to pick up in the next ingest of this document.
