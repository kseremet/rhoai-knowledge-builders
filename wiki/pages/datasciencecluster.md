---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L342-509
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L542-590
verified: 2026-08-25
confidence: documented
---

# DataScienceCluster (DSC)

`DataScienceCluster` (`datasciencecluster.opendatahub.io/v2`) is the custom
resource that tells the [[rhoai-operator]] which RHOAI components to
install and manage. It normally exists as a single cluster instance named
`default-dsc`. Depends on the Operator being installed first (see
[[installing-the-rhoai-operator]]); created/edited via
[[installing-rhoai-components]].

## Namespace / scope

Cluster-scoped custom resource (not namespaced itself), but it drives
workloads into `redhat-ods-applications` (or a custom applications
namespace) and `rhods-notebooks` (or a custom workbench namespace).

## The knob that matters: `spec.components.<name>.managementState`

Each component under `spec.components` takes:

- **`Managed`** — the Operator installs it and actively keeps it running,
  upgrading only when safe.
- **`Removed`** — the Operator does not install it, and removes it if
  already installed.
- **`Unmanaged`** (used for `kueue` specifically, per
  [[installing-distributed-workloads-components]] and
  [[configuring-kueue-workload-management]]) — the Operator does not
  manage the component's lifecycle, letting another Operator (e.g. Red Hat
  build of Kueue) own it. See [[kueue-workload-management]] for day-2
  operation once installed, and
  [[migrating-to-red-hat-build-of-kueue-operator]] for moving off the
  deprecated embedded-Kueue `Managed` state.

Components listed in the example CR: `aipipelines` (with a nested
`argoWorkflowsControllers.managementState` — see
[[configuring-pipelines-with-own-argo-workflows]]), `dashboard`,
`feastoperator`, `kserve` (see [[configuring-kserve-oauth-proxy-sidecar-resources]]
for its `oauthProxy` sub-field), `kueue`, `ogx` (see [[ogx-server]]),
`mlflowoperator`, `modelregistry` (with `registriesNamespace`), `ray`,
`trainingoperator`, `trustyai`, `workbenches` (with `workbenchNamespace` —
see [[configuring-custom-namespaces-for-rhoai]]).

A component key rendered as `component-name: {}` in `spec.components` means
that component is not installed.

## Status

`status.phase` reports `Progressing` → `Ready`. `status.installedComponents`
lists each component with a boolean `true`/`false`. The dashboard becomes
reachable before `Ready`, but not all components are guaranteed ready yet.

## Related object

[[dscinitialization]] — the companion cluster-wide-initialization CR
(trusted CA bundle, operator log level), distinct from DSC (which is purely
about which components to install).

## Forward references / gaps

This guide does not document every component's own configuration surface
(e.g. `feastoperator`, `trustyai`, `mlflowoperator` fields) beyond
management state — those likely have their own component-specific guides
not yet ingested.
