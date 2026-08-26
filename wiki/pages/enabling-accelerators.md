---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L620-641
  - raw/rhoai/3.5/managing-openshift-ai.md#L778-939
verified: 2026-08-25
confidence: documented
---

# Enabling accelerators in RHOAI

Overview of accelerator enablement across vendors. The install guide's own
"Enabling accelerators" section is a thin forward reference; the actual
per-vendor procedures live in the "Managing OpenShift AI" guide's Chapter
7, now ingested as their own pages:

- [[enabling-nvidia-gpus]] — install the upstream NVIDIA GPU Operator (see
  also the ~20 [[gpu-operator-clusterpolicy|GPU Operator component pages]]
  documenting its CRDs), delete the `migration-gpu-status` ConfigMap,
  restart the dashboard.
- [[enabling-intel-gaudi-accelerators]] — install the Intel Gaudi Base
  Operator; **no prebuilt workbench image ships for Gaudi**, so a custom
  one must be built via [[creating-custom-workbench-images]]; optional PID
  limit tuning for heavy multi-accelerator/Ray workloads.
- [[enabling-amd-gpus]] — verify AMD GPU presence via `lspci`/`oc describe
  node`, then install the AMD GPU Operator and its drivers; use the
  [[rhoai-workbench|ROCm workbench images]] once enabled.
- IBM Spyre — "Enabling IBM Spyre accelerators" — **still not ingested into
  this wiki**; neither guide processed so far (install guide or this
  managing guide) documents it.

## Preconditions

- Logged in to the OpenShift cluster with the `cluster-admin` role.
- The accelerator hardware is installed and detected by the cluster.

## Steps

1. Follow the relevant vendor page above.
2. After installing the accelerator, create a hardware profile — "Working
   with hardware profiles" (`working_with_accelerators` guide, still not
   yet ingested into this wiki — referenced by all three vendor pages
   above as their common next step).

## Verification

1. Installed Operators page → confirm both the vendor's accelerator
   Operator and **Node Feature Discovery (NFD)** and **Kernel Module
   Management (KMM)** appear.
2. A few minutes after full install of NFD and the accelerator Operator,
   `oc describe node <node name>` shows the accelerator in node
   `Capacity`/`Allocatable`, e.g. for NVIDIA:
   ```
   Capacity:
     nvidia.com/gpu: 1
   Allocatable:
     nvidia.com/gpu: 1
   ```

## Gap

The `working_with_accelerators` guide (hardware profiles — the step every
vendor page ends on) and IBM Spyre enablement are still not ingested into
this wiki. Everything else this page previously forward-referenced is now
covered by the three vendor pages above.
