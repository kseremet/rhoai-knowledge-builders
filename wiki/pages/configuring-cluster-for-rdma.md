---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1506-1536
verified: 2026-08-25
confidence: documented
---

# Configuring a cluster for RDMA

Sets up NVIDIA GPUDirect RDMA (Remote Direct Memory Access), giving GPUs a
direct interconnect. **As of 2.19**, Red Hat supports this for NVIDIA GPUs
only, over Ethernet or InfiniBand, same-cluster only — see
[[enabling-nvidia-gpus]].

## Preconditions

- OpenShift cluster access as a cluster administrator.
- Multiple worker nodes with supported NVIDIA GPUs, and a compatible NVIDIA
  accelerated networking platform.
- RHOAI installed with the required distributed-training components — see
  [[installing-distributed-workloads-components]].
- Distributed-training resources already configured — see
  [[configuring-quota-management-for-distributed-workloads]].

## Steps

1. Log in to the OpenShift console as a cluster administrator.
2. Enable NVIDIA GPU support (NFD Operator + NVIDIA GPU Operator) — see
   [[enabling-nvidia-gpus]]. **After** the NVIDIA GPU Operator is
   installed, ensure `rdma` is set to `enabled` in the `ClusterPolicy` CR
   (see [[gpu-operator-clusterpolicy]]).
3. Install and configure the NVIDIA Network Operator (to simplify managing
   NVIDIA networking resources) — via "Adding Operators to a cluster" (OCP
   docs, not ingested) for install, and NVIDIA's Network Operator
   Application Notes (external, not ingested) for configuration examples.
4. Optional, for Single Root I/O Virtualization (SR-IOV) deployment modes:
   install and configure the SR-IOV Network Operator (OCP docs, not
   ingested).
5. Increase the pinned-memory limit for non-root users in CRI-O, via the
   Machine Configuration Operator: **Administrator** perspective →
   **Compute → MachineConfigs** → **Create MachineConfig**, e.g.:
   ```yaml
   apiVersion: machineconfiguration.openshift.io/v1
   kind: MachineConfig
   metadata:
     labels:
       machineconfiguration.openshift.io/role: worker
     name: 02-worker-container-runtime
   spec:
     config:
       ignition:
         version: 3.2.0
       storage:
         files:
         - contents:
             inline: |
               [crio.runtime]
               default_ulimits = [
                 "memlock=-1:-1"
               ]
           mode: 420
           overwrite: true
           path: /etc/crio/crio.conf.d/10-custom
   ```
   Adjust `default_ulimits` for your environment, **Create**, then restart
   worker nodes to apply it. This lets non-root users run RDMA training
   jobs under OpenShift's most restrictive default security context.

## Verification

1. **Workloads → Pods** in the relevant project → confirm a pod is running
   for each newly-installed Operator.
2. Confirm RDMA is actually in use: edit the `PyTorchJob` resource to set
   `NCCL_DEBUG=INFO`:
   ```yaml
   spec:
     containers:
     - command:
       - /bin/bash
       - -c
       - "your container command"
       env:
       - name: NCCL_SOCKET_IFNAME
         value: "net1"
       - name: NCCL_IB_HCA
         value: "mlx5_1"
       - name: NCCL_DEBUG
         value: "INFO"
   ```
   Run the job and check pod logs for a line like:
   ```
   NCCL INFO NET/IB : Using [0]mlx5_1:1/RoCE [RO]
   ```
