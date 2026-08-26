---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L778-816
verified: 2026-08-25
confidence: documented
---

# Enabling NVIDIA GPUs in RHOAI

Installs and wires in the upstream NVIDIA GPU Operator (whose CRDs are
documented in this wiki under [[gpu-operator-clusterpolicy]] and its
family of component pages) so RHOAI can schedule workloads onto NVIDIA
GPUs. This is the real procedure that [[enabling-accelerators]] previously
only forward-referenced.

**If running in a disconnected self-managed environment**, use the
disconnected-install guide's own "Enabling accelerators" procedure instead
(not ingested into this wiki) — the steps below assume a connected
cluster.

## Preconditions

- Logged in to the OpenShift cluster with the `cluster-admin` role.
- An NVIDIA GPU is installed and detected in the environment.
- **OpenShift 4.21+ note**: OLMv1 catalog is enabled by default (Technology
  Preview) and may redirect Operator installs (e.g. NFD, NVIDIA GPU
  Operator) to a `ClusterExtensions` page instead of the standard install
  form. Disable the OLMv1 catalog first to get the standard flow — see
  [[troubleshooting-rhoai-operator-installation]].

## Steps

1. Follow NVIDIA's own documentation, "NVIDIA GPU Operator on Red Hat
   OpenShift Container Platform" (external, not ingested), to install the
   Node Feature Discovery (NFD) Operator and the NVIDIA GPU Operator.
   **Important**: after installing NFD, you must create a
   `NodeFeatureDiscovery` instance; after installing the NVIDIA GPU
   Operator, you must create a `ClusterPolicy` populated with default
   values — see [[gpu-operator-clusterpolicy]] for that CRD.
2. Delete the **migration-gpu-status** ConfigMap: OpenShift console,
   **Administrator** perspective, project **All Projects** or
   `redhat-ods-applications` → find the ConfigMap → action menu (⋮) →
   **Delete ConfigMap** → confirm → **Delete**.
3. Restart the dashboard: **Workloads** → **Deployments** → project **All
   Projects** or `redhat-ods-applications` → find `rhods-dashboard` →
   action menu (⋮) → **Restart Rollout** → wait for **Status** to show all
   pods fully restarted.

## Verification

- The reset **migration-gpu-status** instance is gone from the
  `HardwareProfile` CRD's **Instances** tab.
- Installed Operators page (**Ecosystem** → **Installed Operators** on
  OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19) shows
  **NVIDIA GPU**, **Node Feature Discovery (NFD)**, and **Kernel Module
  Management (KMM)**.
- A few minutes after full install, `oc describe node <node name>` shows
  the GPU under `Capacity`/`Allocatable`:
  ```
  Capacity:
    cpu: 4
    ...
    nvidia.com/gpu: 1
    pods: 250
  Allocatable:
    cpu: 3920m
    ...
    nvidia.com/gpu: 1
    pods: 250
  ```

## Notes

- Red Hat supports accelerators within the **same cluster only**.
- **As of 2.19**, Red Hat supports NVIDIA GPUDirect RDMA (over Ethernet or
  InfiniBand) for NVIDIA GPUs only — see [[configuring-cluster-for-rdma]]
  for the cluster-side setup.

## Next step

After installing the Operator, create a hardware profile ("Working with
hardware profiles", not yet ingested into this wiki).
