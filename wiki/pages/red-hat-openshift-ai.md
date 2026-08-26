---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L32-70
verified: 2026-08-25
confidence: documented
---

# Red Hat OpenShift AI

Red Hat OpenShift AI (RHOAI) is an AI/ML platform that provides tools to
train, serve, and monitor machine learning models on-premise, in the public
cloud, or at the edge, deployed on OpenShift. Its purpose is letting data
scientists and MLOps engineers collaborate from experiment to production in
one consistent environment.

**As of 3.5**, RHOAI runs on any supported OpenShift version, including
disconnected environments.

## The data science workflow RHOAI is built around

1. Define the business problem and goals.
2. Gather, clean, and prepare data (often federated from multiple sources).
3. Evaluate and select ML models for the use case.
4. Train models, tuning parameters against training data — typically several
   candidate models compared on tradeoffs like time and memory.
5. Integrate a model into an application: deployment and testing.
6. Monitor and manage deployed models — tracking prediction and performance
   metrics in production.
7. Refine and retrain models based on observed performance.

## Where the pieces live

RHOAI organizes data science work into [[rhoai-project]]s. Within a project
you work in a [[rhoai-workbench]] (an IDE) to develop and train models, then
deploy them for serving, automate retraining with AI pipelines, and monitor
what's running. Terminology used throughout the rest of this wiki for RHOAI
is defined on [[rhoai-glossary]] — read that page once rather than having
each term re-explained per page.

Before any of that, a cluster administrator installs the platform itself —
see [[rhoai-operator]] and [[installing-the-rhoai-operator]] →
[[installing-rhoai-components]] → [[accessing-the-rhoai-dashboard]].

Getting started, in order: [[logging-in-to-openshift-ai]] →
[[creating-a-rhoai-project]] → [[creating-a-workbench]]. From there the guide
points to (not yet ingested into this wiki): working with AI pipelines,
deploying models, working with accelerators (partial — see
[[enabling-accelerators]]), working with distributed workloads (partial —
see [[installing-distributed-workloads-components]]), and working with
connected applications — each is its own RHOAI 3.5 guide in
`raw/rhoai/3.5/`.

## Related but separate products, not RHOAI itself

Several things referenced from RHOAI's own docs are separate products with
their own documentation, not part of RHOAI:

- [[gpu-operator-clusterpolicy]] and its sibling pages — the upstream NVIDIA
  GPU Operator, which RHOAI depends on for GPU support but does not ship.
- The [Red Hat AI Model Optimization Toolkit / LLM Compressor](https://docs.redhat.com/en/documentation/red_hat_ai_inference_server/3.2/html/red_hat_ai_model_optimization_toolkit/supported-model-compression-workflows_llm-compressor)
  — as of RHOAI 2.25, integrated into supported RHOAI workbench images and AI
  pipelines for model compression, but documented under Red Hat AI Inference
  Server, not RHOAI. Not yet ingested into this wiki.
