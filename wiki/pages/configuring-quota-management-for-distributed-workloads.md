---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1267-1505
verified: 2026-08-25
confidence: documented
---

# Configuring quota management for distributed workloads

Creates/verifies the three Kueue resources (`ResourceFlavor`,
`ClusterQueue`, `LocalQueue`) that let several projects share cluster
resources for distributed training/data-processing (`RayJob`, `RayCluster`,
`PyTorchJob`) and other Kueue-managed workloads. Builds on
[[configuring-kueue-workload-management]] and
[[installing-distributed-workloads-components]].

**As of 3.5**, Red Hat supports only NVIDIA and AMD GPU accelerators for
distributed workloads, and does **not** support shared cohorts.

## Preconditions

- Logged in with the `cluster-admin` role.
- OpenShift CLI (`oc`) installed.
- The Red Hat build of Kueue Operator installed and activated — see
  [[configuring-kueue-workload-management]].
- Distributed workloads components installed — see
  [[installing-distributed-workloads-components]].
- A project with a running workbench on a default image containing the
  CodeFlare SDK (e.g. **Standard Data Science**).
- 1.6 vCPU and 2 GiB memory available in addition to base RHOAI resources,
  and the resources physically present in the cluster.
- For GPUs: GPU support enabled — see [[enabling-nvidia-gpus]] or
  [[enabling-amd-gpus]].

## Steps

1. `oc login <openshift_cluster_url> -u <admin_username> -p <password>`
2. **Resource flavor** — check for an existing one (`oc get
   resourceflavors`); edit in place with `oc edit resourceflavor
   <name>` if needed, or create `default_flavor.yaml`:
   ```yaml
   apiVersion: kueue.x-k8s.io/v1beta1
   kind: ResourceFlavor
   metadata:
     name: <example_resource_flavor>
   ```
   Apply with `oc apply -f default_flavor.yaml`.
3. **Cluster queue** — RHOAI auto-creates a default `ClusterQueue` when
   Kueue is activated; check with `oc get clusterqueues`, edit in place
   with `oc edit clusterqueue <name>`, or create `cluster_queue.yaml`:
   ```yaml
   apiVersion: kueue.x-k8s.io/v1beta1
   kind: ClusterQueue
   metadata:
     name: <example_cluster_queue>
   spec:
     namespaceSelector: {} # {} = all namespaces may use this queue
     resourceGroups:
     - coveredResources: ["cpu", "memory", "nvidia.com/gpu"] # amd.com/gpu for AMD
       flavors:
       - name: "<resource_flavor_name>"
         resources:
         - name: "cpu"
           nominalQuota: 9
         - name: "memory"
           nominalQuota: 36Gi
         - name: "nvidia.com/gpu"
           nominalQuota: 5
   ```
   You must specify a quota (even `0`) for every resource users can
   request: include it in `coveredResources`, and give it a `name` +
   `nominalQuota` under `flavors.resources`. The cluster queue only admits
   a workload if its total requested resources fit within these quotas.
   Apply with `oc apply -f cluster_queue.yaml`.
4. **Local queue** — points a project namespace at a cluster queue. If
   Kueue is enabled in the dashboard, new dashboard-created projects may
   already have one. Check with `oc get localqueues -n
   <project_namespace>`, edit with `oc edit localqueue <name> -n
   <project_namespace>`, or create `local_queue.yaml`:
   ```yaml
   apiVersion: kueue.x-k8s.io/v1beta1
   kind: LocalQueue
   metadata:
     name: <example_local_queue>
     namespace: <project_namespace>
   spec:
     clusterQueue: <cluster_queue_name>
   ```
   Apply with `oc apply -f local_queue.yaml`.

## Verification

```
oc get localqueues -n <project_namespace>
```

## Example configurations

Reusable starting points for common scenarios (no shared cohorts, per the
3.5 restriction above).

**NVIDIA-only** — resource flavor + cluster queue per GPU type:
```yaml
apiVersion: kueue.x-k8s.io/v1beta1
kind: ResourceFlavor
metadata:
  name: "a400node"
spec:
  nodeLabels:
    instance-type: nvidia-a400-node
  tolerations:
  - key: "HasGPU"
    operator: "Exists"
    effect: "NoSchedule"
---
apiVersion: kueue.x-k8s.io/v1beta1
kind: ClusterQueue
metadata:
  name: "a400queue"
spec:
  namespaceSelector: {} # match all.
  resourceGroups:
  - coveredResources: ["cpu", "memory", "nvidia.com/gpu"]
    flavors:
    - name: "a400node"
      resources:
      - name: "cpu"
        nominalQuota: 16
      - name: "memory"
        nominalQuota: 64Gi
      - name: "nvidia.com/gpu"
        nominalQuota: 2
```
(An equivalent `a1000node`/`a1000queue` pair follows the same shape.)

**NVIDIA + AMD, separate (non-cohorted) queues per team/vendor**:
```yaml
apiVersion: kueue.x-k8s.io/v1beta1
kind: ResourceFlavor
metadata:
  name: "amd-node"
spec:
  nodeLabels:
    instance-type: amd-node
  tolerations:
  - key: "HasGPU"
    operator: "Exists"
    effect: "NoSchedule"
---
apiVersion: kueue.x-k8s.io/v1beta1
kind: ClusterQueue
metadata:
  name: "team-a-amd-queue"
spec:
  namespaceSelector: {} # match all.
  resourceGroups:
  - coveredResources: ["cpu", "memory", "amd.com/gpu"]
    flavors:
    - name: "amd-node"
      resources:
      - name: "cpu"
        nominalQuota: 16
      - name: "memory"
        nominalQuota: 64Gi
      - name: "amd.com/gpu"
        nominalQuota: 2
```
(An equivalent `nvidia-node`/`team-a-nvidia-queue` pair follows the same
shape, using `nvidia.com/gpu`.)

## See also

[[configuring-cluster-for-rdma]] for RDMA-specific cluster setup on top of
this quota configuration; [[troubleshooting-distributed-workloads-administrators]]
and [[troubleshooting-kueue]] for symptoms tied to misconfigured quotas.
