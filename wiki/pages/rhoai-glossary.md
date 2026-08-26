---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L71-78
verified: 2026-08-25
confidence: documented
---

# RHOAI glossary

Terms as defined by Red Hat's own "Getting started" guide, condensed. Several
of these have (or will get) their own dedicated wiki pages with real
procedural/component detail once their source guide is ingested — this page
is the quick-reference definition, not the full page; follow the
[[wiki-link]] where one exists.

**accelerator** — specialized circuit offloading compute from the CPU; in
deep learning, typically a GPU. See [[gpu-operator-clusterpolicy]] for the
NVIDIA GPU Operator that provisions GPU accelerators in-cluster.

**AI pipelines** — workflow engine for automating pipelines such as model
training/evaluation, including experiment tracking, artifact storage, and
versioning. RHOAI guide `working-with-ai-pipelines.md` not yet ingested.

**bias detection** — calculating fairness metrics to detect AI models
delivering unfair outcomes based on certain attributes.

**custom resource (CR) / custom resource definition (CRD)** — standard
Kubernetes concepts: a CRD defines a new object `Kind`; a CR is an instance
of one.

**connection / connection type** — a project-scoped configuration storing
parameters to connect to an S3-compatible object store, database, or
OCI-compliant container registry. Connection types: OCI-compliant container
registry, S3-compatible object storage, URI.

**project** — see [[rhoai-project]].

**disconnected environment** — a restricted network with no active internet
connection; RHOAI 3.5 supports deployment there (see [[red-hat-openshift-ai]]).

**distributed workloads** — data science workloads run simultaneously across
multiple nodes in an OpenShift cluster. RHOAI guide
`working-with-distributed-workloads.md` not yet ingested.

**fine-tuning** — adapting a pre-trained model to a specific task via
additional training: full fine-tuning (all parameters) or parameter-efficient
fine-tuning (a subset, or new parameters, while freezing the rest).

**graphics processing unit (GPU)** — parallel-processing hardware heavily
used in ML.

**inference / inference server** — using a trained model to generate
predictions from input data; an inference server is what performs that.

**large language model (LLM)** — a language model with a large parameter
count, trained on a large text corpus.

**machine learning (ML)** — the branch of AI using data and algorithms to
imitate how humans learn.

**model** — functions/algorithms trained and tested on a dataset to produce
predictions or decisions.

**model catalog** — curated library for discovering and evaluating available
generative AI models. RHOAI guide `working-with-the-model-catalog.md` and
`manage-and-govern-model-catalog-sources.md` not yet ingested.

**model registry** — central repository of ML model metadata from inception
to deployment (deployment environment, project origin, hyperparameters,
performance metrics, deployment events). RHOAI guides
`working-with-model-registries.md` and `managing-model-registries.md` not yet
ingested.

**model server** — a container hosting a model, exposing an API for
inference and returning predictions.

**model-serving runtime** — a component/framework that helps create model
servers and build inference-optimized APIs.

**MLOps** — collaboration practice between data scientists and operations to
manage the production ML lifecycle (development, training, validation,
deployment, monitoring, management), increasing automation via CI/CD-style
methods.

**notebook interface** — an interactive document combining executable code,
descriptive text, and code output.

**object storage** — storing data as discrete objects in a flat pool, no file
hierarchy — typically used in the cloud.

**OpenShift Container Platform cluster** — the group of machines running the
controllers/pods/services/config needed for containerized apps.

**persistent storage / persistent volume claim (PVC)** — a persistent volume
retaining files/models/artifacts across components (deployments, pipelines,
workbenches); a PVC is a user's request for cluster storage.

**quantization** — compressing foundation model weights to speed up
inference and reduce memory needs.

**serving** — hosting a trained model as a network-accessible service,
answering REST/gRPC inference requests with predictions.

**ServingRuntime** — a CRD defining pod templates that serve one or more
particular model formats: container image, supported formats, other config
via env vars. Dynamically loads/unloads models from disk to memory on demand,
exposing a gRPC endpoint for inference.

**vLLM** — a high-throughput inference engine for LLMs, integrating with
popular models/frameworks.

**workbench / workbench image** — see [[rhoai-workbench]].

**YAML** — the human-readable data-serialization language RHOAI configuration
and CRs are written in.
