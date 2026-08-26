---
type: component
products: [gpu-operator]
versions: []
sources:
  - raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md#L932-1360
verified: 2026-08-25
confidence: documented
---

# NVIDIA Driver (`spec.driver`)

The GPU kernel driver component of the [[gpu-operator-clusterpolicy]] CR — see
that page for the fields (`enabled`, `image`, `repository`, `version`, `env`,
`hostNetwork`, `resources`, ...) this component shares with the operator's
other managed components. Everything else the GPU Operator deploys
([[nvidia-dcgm]], [[nvidia-device-plugin]], [[gpu-feature-discovery]],
[[nvidia-mig-manager]], [[nvidia-gdrcopy]], [[nvidia-gpudirect-storage]],
[[nvidia-cc-manager]], [[nvidia-container-toolkit]]) depends on the driver
being loaded on the node first. [[gpu-operator-validator]] health-checks this
component after it comes up (its `driver` sub-validator, oddly labeled
"Toolkit validator spec" in the upstream schema — see that page).

`spec.driver` is one of the fields `ClusterPolicy.spec` requires to be
present.

## Fields specific to this component

- `kernelModuleType` (enum: `auto` | `open` | `proprietary`, default `auto`) —
  which kernel module flavor to install. `auto` picks based on the GPU devices
  present and the driver branch. Supersedes the deprecated `useOpenKernelModules`
  boolean, which the operator no longer honors.
- `usePrecompiled` (bool) — use pre-compiled kernel modules instead of building
  on the node.
- `useNvidiaDriverCRD` (bool) — delegate driver deployment to the separate
  `NVIDIADriver` CRD instead of managing it directly from `ClusterPolicy`.
- `certConfig.name` — configmap with custom certificates for the driver container.
- `repoConfig.configMapName` — configmap with a custom package repo config for
  the driver container.
- `kernelModuleConfig.name` — configmap with kernel module config parameters.
- `secretEnv` — name of a Secret providing secret environment variables to the
  driver container (parallel to the plain `env` list).
- `licensingConfig` — for NVIDIA vGPU licensing: `nlsEnabled` (bool, use NVIDIA
  Licensing System), `secretName` (secret holding the licensing token);
  `configMapName` is deprecated in favor of `secretName`.
- `rdma` (`GPUDirectRDMASpec`) — `enabled` turns on GPUDirect RDMA via the
  operator; `useHostMofed` uses MOFED drivers already installed on the host
  instead of the operator-managed ones.
- `virtualTopology.config` — configmap name for the vGPU virtual topology
  daemon's `nvidia-topologyd.conf`.
- `manager` — spec for the separate "NVIDIA Driver Manager" initContainer
  (its own `image`/`repository`/`version`/`env`/`imagePullPolicy`/`imagePullSecrets`).
  Two more independent copies of this same pattern exist on [[nvidia-vfio-manager]]
  and [[nvidia-vgpu-manager]] (both also called `driverManager`) — they are not
  shared with this one.
- `livenessProbe`, `readinessProbe`, `startupProbe` — standard Kubernetes probe
  tuning (`failureThreshold`, `initialDelaySeconds`, `periodSeconds`,
  `successThreshold`, `timeoutSeconds`) for the driver container.
- `upgradePolicy` — auto-upgrade behavior for the driver across nodes:
  - `autoUpgrade` (bool, default `false`) — master switch; when `false` every
    other field below is ignored.
  - `maxParallelUpgrades` (default `1`, `0` = unlimited) — how many nodes
    upgrade at once.
  - `maxUnavailable` (int-or-string, default `25%`) — max nodes with the
    driver unavailable during upgrade.
  - `drain` — node drain behavior during upgrade: `enable`, `force`,
    `deleteEmptyDir`, `podSelector`, `timeoutSeconds` (default `300`).
  - `podDeletion` — deletion behavior for pods holding special resources
    during upgrade: `force`, `deleteEmptyDir`, `timeoutSeconds` (default `300`).
  - `waitForCompletion` — wait for jobs matching `podSelector` to finish
    before upgrading, with its own `timeoutSeconds` (default `0` = infinite).
