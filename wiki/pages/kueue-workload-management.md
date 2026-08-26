---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L940-1052
verified: 2026-08-25
confidence: documented
---

# Kueue workload management in RHOAI

Kueue manages AI/ML workload scale-out via hierarchical quota management,
dynamic resource allocation, and prioritized job scheduling — preventing
cluster contention, ensuring fair access across teams, and maximizing use
of heterogeneous compute (e.g. accelerators). It schedules `RayJob`,
`RayCluster`, `PyTorchJob` (distributed training), `Notebook` (workbenches),
and `InferenceService` (model serving). Validation/enforcement apply only
in namespaces labeled `kueue.openshift.io/managed=true`.

**Breaking change, as of 2.24**: the embedded Kueue component is
deprecated. Kueue is now provided by the separately-installed **Red Hat
build of Kueue Operator**; running both the embedded Kueue and this
Operator on the same cluster is unsupported (conflicting controllers manage
the same resources). RHOAI does **not** auto-migrate existing workloads —
see [[migrating-to-red-hat-build-of-kueue-operator]].

Distinct from [[installing-distributed-workloads-components]] (the
install-guide procedure that gets `kueue`/`ray`/`trainingoperator` onto the
`DataScienceCluster` in the first place): this page and its siblings
([[configuring-kueue-workload-management]],
[[troubleshooting-kueue]]) cover the day-2 operation of Kueue once
installed.

## Kueue management states

Set via `spec.components.kueue.managementState` on the
[[datasciencecluster]] CR. Only `Unmanaged` is supported for using Kueue
with RHOAI: RHOAI integrates with an existing Kueue installation owned by
the Red Hat build of Kueue Operator (which must already be installed and
running). Enabling `Unmanaged` mode makes the RHOAI Operator create a
default `Kueue` CR if one doesn't already exist, which prompts the Red Hat
build of Kueue Operator to activate Kueue.

## Queue enforcement for projects

Enabling a project for Kueue management means labeling its namespace
`kueue.openshift.io/managed=true` (the legacy `kueue-managed` label still
works for backward compatibility, but the new label is recommended). Once
labeled, a validating webhook requires the `kueue.x-k8s.io/queue-name`
label on any new/updated `InferenceService`, `Notebook`, `PyTorchJob`,
`RayCluster`, or `RayJob` in that namespace — missing the label blocks the
create/update. (This webhook replaces the Validating Admission Policy the
deprecated embedded Kueue used.) You can apply hardware profiles to other
workload types too, but the webhook only enforces the queue-name label for
these five resource kinds.

RHOAI auto-creates a default, cluster-scoped `ClusterQueue` (if none
exists) and a namespace-scoped `LocalQueue` for the namespace (if none
exists), both annotated `opendatahub.io/managed=false` — so cluster
administrators can freely modify or delete them post-creation.

## Restrictions

- Namespaces need `kueue.openshift.io/managed=true` for Kueue validation
  and queue enforcement to apply.
- Dashboard-created workloads (workbenches, model servers) must use a
  hardware profile that specifies a local queue — RHOAI then automatically
  applies the matching `kueue.x-k8s.io/queue-name` label.
- Hardware profiles with node selectors or tolerations cannot be used
  under Kueue — to pin workloads to specific nodes, use a hardware profile
  whose local queue is tied to a queue configured with the right resource
  flavors instead.
- Workbenches are not suspendable, so they can only be assigned to a local
  queue backed by a **non-preemptive** cluster queue — RHOAI's
  auto-created default cluster queue is non-preemptive.

## Workflow across roles

- **Cluster administrator**: installs the Red Hat build of Kueue Operator,
  activates the integration (`managementState: Unmanaged`), configures
  quotas, and enables Kueue in the dashboard (`disableKueue: false` on
  `OdhDashboardConfig`) — see [[configuring-kueue-workload-management]].
  Enabling Kueue in the dashboard auto-enables Kueue management for **new**
  dashboard-created projects only; existing projects, or projects created
  via `oc`, need the namespace label applied manually.
- **RHOAI administrator**: creates Kueue-enabled hardware profiles so users
  can submit workloads from the dashboard ("Working with hardware
  profiles", not yet ingested).
- **ML engineer / data scientist**: selects a Kueue-enabled hardware
  profile when creating dashboard workloads; for CLI/SDK-created workloads
  (e.g. distributed training jobs), adds the `kueue.x-k8s.io/queue-name`
  label pointing at the target `LocalQueue`.

## See also

[[troubleshooting-kueue]] for common Kueue-workload error symptoms.
