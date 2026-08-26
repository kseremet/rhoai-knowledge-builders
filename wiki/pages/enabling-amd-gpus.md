---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L872-939
verified: 2026-08-25
confidence: documented
---

# Enabling AMD GPUs in RHOAI

Integrates AMD GPUs for AI/ML workloads via two pieces: the [[rhoai-workbench|
ROCm workbench images]] (already listed as provided images — CUDA-equivalent
for AMD, with PyTorch/TensorFlow variants), and the AMD GPU Operator, which
automates driver install, device-plugin setup, and node labeling.

## Verifying AMD GPU availability (before installing the Operator)

**Preconditions**: administrative access to the OpenShift cluster; a node
with an AMD GPU; `oc` and node terminal access.

1. Confirm GPU resources are allocatable:
   - `oc get nodes` — identify the node with the AMD GPU.
   - `oc describe node <node_name>` — confirm `amd.com/gpu` appears under
     **Capacity** and **Allocatable**, e.g.:
     ```
     Capacity:
       amd.com/gpu: 1
     Allocatable:
       amd.com/gpu: 1
     ```
2. Check the PCI device directly: `oc debug node/<node_name> chroot /host`,
   then
   ```
   lspci | grep -E "MI210|MI250|MI300|MI350|MI355"
   ```
   Expect a line like `03:00.0 Display controller: Advanced Micro Devices,
   Inc. [AMD] Instinct MI210`.
3. Optional, if the ROCm stack is already on the node: `rocminfo` should
   report compute units, memory, and driver status.

**Verification**: `amd.com/gpu` present in node Capacity/Allocatable;
`lspci` matches a supported AMD model; `rocminfo` (if run) confirms driver
status.

## Enabling AMD GPUs

**Preconditions**: logged in to OpenShift with the `cluster-admin` role; an
AMD GPU installed and detected; if running in a public cloud, confirmed
that the provider offers instances with AMD GPU + ROCm support (check the
AMD GPU Operator support matrix).

1. Install the latest AMD GPU Operator (external AMD documentation, not
   ingested) — via the AMD documentation, or alternatively from the Red
   Hat Catalog.
2. Configure the AMD drivers the Operator requires (external AMD
   documentation, not ingested).

After installing the Operator, create a hardware profile ("Working with
hardware profiles", not yet ingested).

## Verification

- Installed Operators page (**Ecosystem** → **Installed Operators** on
  OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19) lists
  **AMD GPU Operator**, **Node Feature Discovery (NFD)**, and **Kernel
  Module Management (KMM)**.

**Note**: incorrect driver installation/configuration can prevent the AMD
GPU from being recognized or functioning — follow all steps carefully.
