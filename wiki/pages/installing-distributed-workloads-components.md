---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L542-590
verified: 2026-08-25
confidence: documented
---

# Installing the distributed workloads components

Enables RHOAI's distributed workloads feature (multi-node training/data
processing) by setting the right `managementState` values on the
[[datasciencecluster]] CR for `kueue`, `ray`, `trainingoperator`,
`aipipelines`, `workbenches`, and `dashboard`.

## Preconditions

- Logged in with `cluster-admin`, access to the data science cluster.
- RHOAI is installed (see [[installing-rhoai-components]]).
- The **Red Hat build of Kueue Operator** is installed (external doc, not
  ingested).
- Sufficient resources: 1.6 vCPU and 2 GiB memory *in addition to* the
  baseline RHOAI resource requirements (see
  [[installing-the-rhoai-operator]] preconditions).
- The **cert-manager Operator** is installed via the web console (external
  doc, not ingested).
- Optional, for GPUs: NVIDIA or AMD GPU support enabled (see
  [[enabling-nvidia-gpus]] / [[enabling-amd-gpus]]; NVIDIA-specific RDMA
  support is available as of RHOAI 2.19 — GPUDirect RDMA over Ethernet or
  InfiniBand, for same-cluster accelerators only — see
  [[configuring-cluster-for-rdma]]).
- Optional, for self-signed certificates: added to a central CA bundle (see
  [[how-rhoai-handles-certificates]]) — no further config needed;
  distributed-workload pods automatically get the bundle mounted at
  `/etc/pki/tls/certs/odh-trusted-ca-bundle.crt` and
  `/etc/ssl/certs/odh-trusted-ca-bundle.crt` (cluster-wide) or
  `/etc/pki/tls/certs/odh-ca-bundle.crt` and
  `/etc/ssl/certs/odh-ca-bundle.crt` (custom).

## Steps

1. Log in to the web console as a cluster administrator.
2. Installed Operators → **Red Hat OpenShift AI** → **Data Science
   Cluster** tab → `default-dsc` → **YAML** tab.
3. Set `managementState` per the table below, depending on whether the
   workload runs from pipelines, workbenches, or both:

   | Component | Pipelines only | Workbenches only | Pipelines and workbenches |
   |---|---|---|---|
   | `dashboard` | `Managed` | `Managed` | `Managed` |
   | `aipipelines` | `Managed` | `Removed` | `Managed` |
   | `kueue` | `Unmanaged` | `Unmanaged` | `Unmanaged` |
   | `ray` (optional, for Ray-based tuning) | `Managed` | `Managed` | `Managed` |
   | `trainingoperator` (optional, for Kubeflow Training Operator-based tuning) | `Managed` | `Managed` | `Managed` |
   | `workbenches` | `Removed` | `Managed` | `Managed` |

   `kueue` is set `Unmanaged` specifically so the separately-installed Red
   Hat build of Kueue Operator manages it instead of the RHOAI Operator.
4. **Save**. Components in `Managed` state become ready after a short time.

## Verification

Check pod status for **kubeflow-training-operator** and
**kuberay-operator** (project `redhat-ods-applications`), and
**kueue-controller-manager** and **openshift-kueue-operator** (project
`openshift-kueue-operator`):

1. Workloads → Deployments → search by the deployment name.
2. Open the deployment → **Pods** tab → confirm pod status is **Running**.
3. For more detail, open a pod → **Logs** tab.

## Next step

Configure the feature itself — day-2 operation of Kueue and distributed
workloads is covered by [[kueue-workload-management]],
[[configuring-kueue-workload-management]], and
[[configuring-quota-management-for-distributed-workloads]].
