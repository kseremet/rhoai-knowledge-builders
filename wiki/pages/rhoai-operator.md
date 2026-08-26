---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L39-59
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L257-341
verified: 2026-08-25
confidence: documented
---

# Red Hat OpenShift AI Operator

The Red Hat OpenShift AI Operator is a **meta-operator**: it deploys and
maintains all components and sub-operators that make up [[red-hat-openshift-ai]].
It lives at the management layer, below the service-layer pieces
(dashboard, model serving, AI pipelines, Jupyter/[[rhoai-workbench]],
distributed workloads, RAG/[[ogx-server]]) that it installs and manages.

**As of 3.5**, RHOAI is an Operator available self-managed (on OpenShift
Container Platform) or in Red Hat-managed cloud environments (OpenShift
Dedicated with a CCS subscription, ROSA classic/HCP, Azure Red Hat
OpenShift). Only one instance of the Operator is supported per cluster, and
it is not supported alongside the RHOAI Add-on on the same cluster.

## Namespaces it creates

When installed with predefined (default) namespaces:

- `redhat-ods-operator` — contains the Operator itself.
- `redhat-ods-applications` — dashboard and other required OpenShift AI
  components.
- `rhods-notebooks` — where basic workbenches are deployed by default.

Custom namespaces can replace these — see
[[configuring-custom-namespaces-for-rhoai]]. Do not rename or reuse the
predefined namespaces (they must contain `openshift` or `redhat` in the
name), and do not install unrelated ISV applications into RHOAI namespaces.

## Installation objects

Installing the Operator itself (distinct from installing RHOAI
*components*) creates, in the operator namespace:

- A `Namespace` (if not already created).
- An `OperatorGroup` (`operators.coreos.com/v1`).
- A `Subscription` (`operators.coreos.com/v1alpha1`) — specifies the
  [[rhoai-operator-update-channels|update channel]] and, optionally, a
  pinned `startingCSV`.

See [[installing-the-rhoai-operator]] for the runnable procedure.

## What it manages once installed

Once running, the Operator watches a [[datasciencecluster]] custom resource
to decide which RHOAI components to install/manage, and a
[[dscinitialization]] custom resource for cluster-wide initialization
concerns (trusted CA bundle, log level). See those pages for the CRs
themselves.

## Logs and diagnostics

Operator pods run with label `name=rhods-operator` in the operator
namespace (default `redhat-ods-operator`). See
[[configuring-the-rhoai-operator-logger]] for changing log verbosity and
streaming logs, and [[troubleshooting-rhoai-operator-installation]] /
[[rhoai-dashboard-not-accessible]] for common failure symptoms surfaced in
these pods.

## Uninstalling

See [[uninstalling-rhoai]] for the supported CLI removal procedure and what
it does and does not delete.
