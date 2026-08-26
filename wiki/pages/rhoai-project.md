---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L124-154
verified: 2026-08-25
confidence: documented
---

# RHOAI project

In OpenShift, a project is a Kubernetes namespace with additional
annotations, and is the main way user access to resources is managed. In
RHOAI a project is where a data scientist's work is organized and where
other users in the org can collaborate.

A project can contain:

- Connections — access data (e.g. an S3-compatible bucket) without
  hardcoding endpoints/credentials.
- [[rhoai-workbench]]s — for working with data and developing models.
- Deployed models — testable and integratable into applications via API,
  once deployed.
- Pipelines — for automating the ML workflow.

Created via the procedure on [[creating-a-rhoai-project]].
