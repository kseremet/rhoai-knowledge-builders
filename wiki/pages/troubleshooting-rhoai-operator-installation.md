---
type: error
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1312-1541
verified: 2026-08-25
confidence: documented
---

# Troubleshooting RHOAI Operator installation problems

Runbook of distinct installation-time symptoms for [[rhoai-operator]], each
keyed by the verbatim UI behavior or error text you'll actually see. Most
diagnosis paths share a starting point: **Workloads → Pods → set Project to
All Projects or `redhat-ods-operator` → find the `rhods-operator-<random
string>` pod with an error Status → Logs tab → select `rhods-operator`**.
See also [[configuring-the-rhoai-operator-logger]] to raise verbosity
before re-attempting, and [[rhoai-dashboard-not-accessible]] for the
separate post-install "dashboard unreachable" symptom. If a problem isn't
covered here or in the release notes, contact
[Red Hat Support](https://access.redhat.com/support) with a `must-gather`
capture (see [Must-Gather for Red Hat OpenShift AI](https://access.redhat.com/solutions/7061604)).

## Symptom: Operator/dependency install redirects to a ClusterExtensions page

**Verbatim**: clicking **Install** on an Operator tile in OperatorHub (e.g.
Node Feature Discovery, NVIDIA GPU Operator) opens a `ClusterExtensions`
page instead of the normal install form.

**Cause**: on OpenShift 4.21+, the OLMv1 catalog Technology Preview is
enabled by default, changing the install UX for CSV-based (OLMv0)
Operators. When active: `ClusterExtension` CRs must be created manually via
YAML to install through OLMv1, and OLMv0-installed Operators (including
RHOAI and its dependencies) stop appearing under **Ecosystem → Installed
Operators**, even though `oc get csv` still lists them correctly.

**Diagnosis**:
1. OperatorHub → search "Node Feature Discovery" or "NVIDIA GPU Operator".
2. Click the tile → **Install**.
3. Redirected to `ClusterExtensions` instead of the standard form →
   confirms OLMv1 catalog is enabled.

**Fix** — disable the OLMv1 catalog (either method):
- *User Preferences*: username (top right) → **User Preferences** → under
  **OLMv1 Catalog**, clear **Enable OLMv1 catalog**.
- *Software Catalog filter*: **Operators → OperatorHub** → **Types**
  sidebar → **Operators** → clear **Enable OLMv1 Tech Preview**.

Then reinstall the dependency Operator via the now-standard flow; it will
install via OLMv0 and appear correctly under **Installed Operators**.

**Workaround if you must keep OLMv1 on**: view OLMv0-installed Operators
directly at `/k8s/ns/<namespace>/clusterserviceversions`, e.g.
`/k8s/ns/openshift-nfd/clusterserviceversions` or
`/k8s/ns/nvidia-gpu-operator/clusterserviceversions`.

## Symptom: `Failure to pull from quay` retrieving the Operator image

**Cause**: image registry unavailable, network connectivity issue, or the
cluster itself is not operational.

**Diagnosis**: check the **Events** section in OpenShift for detail around
the `Failure to pull from quay` message.

**Fix**: contact Red Hat support.

## Symptom: `ERROR: Deploying on $infrastructure, which is not supported. Failing Installation`

**Cause**: attempting install on infrastructure not documented as
supported by the RHOAI Operator.

**Diagnosis**: Administrator perspective → Workloads → Pods → Project =
All Projects or `redhat-ods-operator` → open the errored
`rhods-operator-<random string>` pod → **Logs** → `rhods-operator` →
confirm the exact error text above.

**Fix**: verify the target environment against [Supported Configurations
for 3.x](https://access.redhat.com/articles/rhoai-supported-configs-3.x)
before attempting a fresh install on a supported environment.

## Symptom: `ERROR: Attempt to create the ODH CR failed.`

**Cause**: unknown — occurs during install, the OpenShift AI CR fails to
be created.

**Diagnosis**: same operator-pod-log path as above; confirm this exact
error text.

**Fix**: contact Red Hat support.

## Symptom: `ERROR: Attempt to create the RHODS Notebooks CR failed.`

**Cause**: unknown — occurs during install, the Notebooks CR fails to be
created.

**Diagnosis**: same operator-pod-log path.

**Fix**: contact Red Hat support.

## Symptom: reinstall via CLI fails with `unable to find DSCInitialization`

**Verbatim** (in `rhods-operator` pod logs): something like
`{"name":"auth"},"namespace":"","name":"auth","reconcileID":"...","error":"unable to find DSCInitialization","stacktrace":"sigs.k8s.io/controller-runtime/pkg/internal/controller.`

**Cause**: the `Auth` custom resource from a previous install was not
deleted before uninstalling and reinstalling the Operator — see
[[uninstalling-rhoai]] for what a proper uninstall removes.

**Diagnosis**: same operator-pod-log path; look for the `unable to find
DSCInitialization` text specifically, tied to reconciling `auth`.

**Fix**:
1. Uninstall the Operator (see [[uninstalling-rhoai]]).
2. Delete the leftover `Auth` CR manually: Administrator perspective →
   **API Explorer** → All groups → `services.platform.opendatahub.io` →
   **Auth** kind → **Instances** tab → action menu (⋮) → **Delete Auth** →
   confirm.
3. Reinstall the Operator (see [[installing-the-rhoai-operator]]).

## Symptom: `ERROR: Attempt to create the RBAC policy for dedicated admins group in $target_project failed.`

**Cause**: unknown — the `dedicated-admins` RBAC policy fails to be
created in the target project.

**Diagnosis**: same operator-pod-log path.

**Fix**: contact Red Hat support.

## Symptom: `ERROR: Addon managed odh parameter secret does not exist.`

**Cause**: an issue in the Operator's install flow prevents creation of the
ODH parameter secret.

**Diagnosis**: same operator-pod-log path.

**Fix**: contact Red Hat support.

## How to tell these apart, quickly

All eight symptoms share the same diagnostic entry point (errored
`rhods-operator-*` pod → Logs). The differentiator is always the exact
`ERROR:` (or, for OLMv1, the UI redirect) text — grep the pod logs for the
verbatim strings above rather than guessing from vague "install failed"
reports. Only two of the eight (OLMv1 redirect, unsupported-infrastructure,
and the DSCInitialization reinstall failure) have a documented root cause
and self-service fix in this guide; the rest ("ODH CR failed", "RHODS
Notebooks CR failed", "RBAC policy ... failed", "odh parameter secret does
not exist") are explicitly documented as occurring "in unknown
circumstances" with resolution = contact support — that's a real
information gap in the source guide, not an omission in this page.
