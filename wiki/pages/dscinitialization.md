---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L656-692
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1231-1270
verified: 2026-08-25
confidence: documented
---

# DSCInitialization (DSCI)

`DSCInitialization` (`dscinitialization.opendatahub.io/v2`) is the
cluster-wide initialization custom resource for the [[rhoai-operator]],
normally a single instance named `default-dsci`. Distinct from
[[datasciencecluster]] (which selects components); DSCI controls two
things: the trusted CA bundle, and the Operator's own log level.

## Namespace / scope

Cluster-scoped. Managed under the Operator's **DSC Initialization** tab in
the web console.

## Knobs that matter

- **`spec.trustedCABundle.managementState`**: `Managed` (default) |
  `Unmanaged` | `Removed`. Controls whether the Operator maintains the
  `odh-trusted-ca-bundle` ConfigMap across namespaces. See
  [[how-rhoai-handles-certificates]] for the full mechanism, and
  [[managing-certificates-without-the-rhoai-operator]] /
  [[removing-the-ca-bundle]] for the procedures that flip this.
- **`spec.trustedCABundle.customCABundle`**: PEM-encoded custom
  certificate(s) added on top of the cluster-wide bundle. See
  [[adding-certificates-to-custom-ca-bundle]].
- **`spec.devFlags.logLevel`**: `error` | `info` (default) | `debug` | a
  custom non-negative integer for finer-grained tracing. Applies only to
  the `rhods-operator` pod itself, not the components it manages. See
  [[configuring-the-rhoai-operator-logger]].

## What depends on it

- The `odh-trusted-ca-bundle` ConfigMap that gets propagated into every
  non-reserved namespace (mounted by workbenches, pipeline pods,
  distributed-workload pods, and referenced by [[ogx-server]]).
- The `rhods-operator` pod's log verbosity.

## Diagnosing DSCI problems

A reinstall failing with `unable to find DSCInitialization` in operator
logs is a known symptom — see
[[troubleshooting-rhoai-operator-installation]] (reinstall-after-uninstall
section).
