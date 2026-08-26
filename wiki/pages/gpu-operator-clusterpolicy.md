---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L32-59
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L151-173
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L174-481
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L1535-1558
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2146-2163
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2244-2272
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L2927-3002
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L3003-3009
verified: 2026-08-25
confidence: documented
---

# GPU Operator ClusterPolicy

`ClusterPolicy` is the CRD the [NVIDIA GPU Operator](https://github.com/NVIDIA/gpu-operator)
watches to decide which of its managed components to deploy and how to configure
them: `apiVersion: apiextensions.k8s.io/v1`, `group: nvidia.com`, `kind:
ClusterPolicy`, `scope: Cluster` — a single cluster-scoped custom resource, version
`v1`, `served: true`, `storage: true`, with a `status` subresource. Its short
output (`oc get clusterpolicy`) shows a `Status` column from `.status.state`
and an `Age` column from `.metadata.creationTimestamp`.

`spec` has one block per managed component (`spec.driver`, `spec.dcgm`,
`spec.devicePlugin`, ...). This page indexes that structure; field detail for
each managed component lives on its own page, linked below. `sources.yaml` in
this wiki's build config tracks the GPU Operator source at `ref: main` (rolling
upstream, no version pin) — there is no fixed product version to record in this
page's frontmatter, unlike RHOAI/OCP guides.

**Not itself an RHOAI component.** RHOAI relies on the GPU Operator to make
GPUs schedulable, but the GPU Operator is developed and shipped upstream by
NVIDIA, not by Red Hat.

**A second, newer CRD exists for the same operator**: see
[[gpu-operator-gpucluster]] (`GPUCluster`, group `nvidia.com`, kind
`GPUCluster`) — a DRA-based (Dynamic Resource Allocation) alternative to
`ClusterPolicy` that explicitly does *not* manage the driver or device plugin
itself. Do not conflate the two; they are separate CRDs from the same
operator, not versions of one CRD.

`spec` requires `daemonsets`, `dcgm`, `dcgmExporter`, `devicePlugin`, `driver`,
`gfd`, `nodeStatusExporter`, `operator`, and `toolkit` to be present (even if
individually disabled) — everything else in the list below is optional.

## Managed components

Each of these is a distinct thing that gets deployed into the cluster (a
DaemonSet or Deployment) when its `spec.<field>.enabled` is true:

- [[nvidia-driver]] — GPU kernel driver (`spec.driver`, required)
- [[nvidia-container-toolkit]] — NVIDIA Container Toolkit (`spec.toolkit`, required)
- [[nvidia-dcgm]] — DCGM hostengine (`spec.dcgm`, required)
- [[nvidia-dcgm-exporter]] — DCGM Prometheus exporter (`spec.dcgmExporter`, required)
- [[nvidia-device-plugin]] — Kubernetes device plugin (`spec.devicePlugin`, required)
- [[gpu-feature-discovery]] — node labeling for GPU capabilities (`spec.gfd`, required)
- [[node-status-exporter]] — reports node readiness for GPU operands (`spec.nodeStatusExporter`, required)
- [[gpu-operator-controller]] — the operator's own controller-manager pod config (`spec.operator`, required)
- [[gpu-operator-validator]] — operator-validator daemonset, health-checks the other operands (`spec.validator`)
- [[nvidia-mig-manager]] — MIG partitioning (`spec.mig`, `spec.migManager`)
- [[nvidia-gdrcopy]] — GDRCopy driver (`spec.gdrcopy`)
- [[nvidia-gpudirect-storage]] — GPUDirect Storage, experimental (`spec.gds`)
- [[nvidia-cc-manager]] — Confidential Computing mode manager (`spec.ccManager`)
- [[nvidia-sandbox-device-plugin]] — device plugin for VM/Kata sandbox workloads (`spec.sandboxDevicePlugin`)
- [[nvidia-vfio-manager]] — VFIO-PCI manager for GPU passthrough to VMs (`spec.vfioManager`)
- [[nvidia-vgpu-manager]] — NVIDIA vGPU host manager (`spec.vgpuManager`)
- [[nvidia-vgpu-device-manager]] — vGPU device/profile configuration (`spec.vgpuDeviceManager`)
- [[kata-manager]] — Kata runtime class provisioning, **deprecated** (`spec.kataManager`)
- [[kata-sandbox-device-plugin]] — device plugin for Kata sandboxes (`spec.kataSandboxDevicePlugin`, distinct from [[nvidia-sandbox-device-plugin]] above)

## Common per-component fields

Nearly every `spec.<component>` block above shares this shape (representative
example: `spec.ccManager`, `#L63-150`):

- `enabled` (bool) — whether the operator deploys this component
- `image`, `repository`, `version` — image name/repo/tag, kept separate so the
  tag can be pinned independently; **do not quote a specific tag here**, it is a
  moving value (CLAUDE.md rule 2)
- `imagePullPolicy`, `imagePullSecrets`
- `args`, `env` (list of name/value pairs)
- `hostNetwork` (bool) — present on most but not all components; absent on
  `gdrcopy` and `gds`
- `resources.limits` / `resources.requests` — standard Kubernetes quantities

Sub-pages below document only the fields a component adds on top of this
common shape.

## Cluster-wide config blocks

These exist in `spec` but are not themselves deployed things — they configure
how other components behave — so they don't get their own page (CLAUDE.md rule
5: one page per *thing*).

**`spec.cdi`** (`#L151-173`) — Container Device Interface settings.
`enabled` (default `true`) controls whether CDI is the mechanism used to make
GPUs accessible to containers; `nriPluginEnabled` (default `false`) runs an NRI
plugin to inject CDI devices into GPU-management containers; `default` is
deprecated and no longer used.

**`spec.daemonsets`** (`#L174-481`) — defaults applied to every DaemonSet the
operator creates: `annotations`, `labels`, `podSecurityContext` (the standard
Kubernetes pod security context — AppArmor/seccomp/SELinux profiles, fsGroup,
run-as-user, sysctls, Windows options), `priorityClassName`, `tolerations`
(standard Kubernetes tolerations), `rollingUpdate.maxUnavailable`, and
`updateStrategy` (`RollingUpdate` default, or `OnDelete`). The same schema is
reused verbatim for `spec.daemonsets` on [[gpu-operator-gpucluster]].

**`spec.hostPaths`** (`#L1535-1558`) — host filesystem paths the operator's
components need: `driverInstallDir`, `kubeletRootDir` (default
`/var/lib/kubelet`), and `rootFS` — the chroot-able host root filesystem path
used by components (MIG Manager, Toolkit Container) that need to
stop/start/restart systemd services on the host.

**`spec.psa`** (`#L2146-2153`) — `enabled` (bool): turn on
PodSecurityAdmission configuration for all pods the operator creates.

**`spec.psp`** (`#L2154-2163`) — **deprecated**: "Pod Security Policies are no
longer supported. Please use PodSecurityAdmission instead." Superseded by
`spec.psa`.

**`spec.sandboxWorkloads`** (`#L2244-2272`) — controls whether the operator
manages the additional operands sandbox/VM workloads need ([[nvidia-vfio-manager]],
[[nvidia-vgpu-manager]], and the sandbox device plugins): `enabled` (bool),
`defaultWorkload` (enum `container` | `vm-passthrough` | `vm-vgpu`, default
`container`) — the default GPU workload type configured on worker nodes, and
`mode` (enum `kubevirt` | `kata`, default `kubevirt`) — which sandbox
technology is in use.

## Status schema

`status` (`#L2927-3002`) — `state` (required; enum `ignored` | `ready` |
`notReady`) is what the `Status` printer column shows; `namespace` records
which namespace the operator is installed in; `conditions` follows the
standard Kubernetes `Condition` shape (`type`, `status`, `reason`, `message`,
`lastTransitionTime`, `observedGeneration`).
