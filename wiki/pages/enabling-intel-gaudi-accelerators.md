---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L817-871
verified: 2026-08-25
confidence: documented
---

# Enabling Intel Gaudi AI accelerators in RHOAI

Integrates Intel Gaudi AI accelerators (up to Gaudi 3, which Red Hat
supports) so data scientists can use Gaudi libraries/software through
custom-configured workbenches. Suited to high-performance deep learning:
Gaudi 3 in particular offers improved training throughput (tensor
processing cores, higher memory bandwidth), lower power draw at scale, and
a scalable multi-node architecture. Unlike NVIDIA (see
[[enabling-nvidia-gpus]]), there is **no prebuilt Gaudi workbench image** —
one must be built via [[creating-custom-workbench-images]].

Three things are required before Gaudi accelerators are usable:

1. Install the latest Intel Gaudi Base Operator.
2. Create/configure a custom workbench image for Gaudi (no prebuilt one
   ships with RHOAI).
3. Manually define a hardware profile for each Gaudi device.

On AWS, the OpenShift platform must support EC2 DL1 instances to use Gaudi
in that environment.

## Preconditions

- Logged in to OpenShift with the `cluster-admin` role.
- The Gaudi accelerator is installed and detected.
- If on AWS: EC2 DL1 instance support confirmed.
- OpenShift CLI (`oc`) installed.

## Steps

1. Install the latest Intel Gaudi Base Operator (external Habana
   documentation, not ingested).
2. **PID-limit tuning, if needed**: OpenShift's default per-pod PID limit
   is 4096. Workloads needing more (multiple Gaudi accelerators, or vLLM
   with Ray) can hit `Resource temporarily unavailable` from PID
   exhaustion. Red Hat recommends 32768 (values over 20000 are sufficient):
   - Label the target node: `oc label node <node_name> custom-kubelet=set-pod-pid-limit-kubelet`
   - Optional: mark the node unschedulable and drain it before this step.
   - Create `custom-kubelet-pidslimit.yaml`:
     ```yaml
     apiVersion: machineconfiguration.openshift.io/v1
     kind: KubeletConfig
     metadata:
       name: custom-kubelet-pidslimit
     spec:
       kubeletConfig:
         PodPidsLimit: 32768
       machineConfigPoolSelector:
         matchLabels:
           custom-kubelet: set-pod-pid-limit-kubelet
     ```
   - Apply it: `oc apply -f custom-kubelet-pidslimit.yaml` — **this reboots
     the node**.
   - If previously marked unschedulable, allow scheduling again after
     reboot.
3. Build a custom workbench image for Gaudi — see
   [[creating-custom-workbench-images]].
4. After installing the Gaudi Base Operator, create a hardware profile
   ("Working with hardware profiles", not yet ingested).

## Verification

- Installed Operators page (**Ecosystem** → **Installed Operators** on
  OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19) lists
  **Intel Gaudi Base Operator**, **Node Feature Discovery (NFD)**, and
  **Kernel Module Management (KMM)**.
