---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L155-217
  - raw/rhoai/3.5/managing-openshift-ai.md#L288-471
verified: 2026-08-25
confidence: documented
---

# RHOAI workbench

An isolated environment inside a [[rhoai-project]] for developing and
experimenting with ML models — writing code to process data, training
models. Not required for every task (e.g. just serving an existing model
doesn't need one), but needed for most data-science workflow tasks.

A workbench is a server-client architecture: the IDE server runs in a
container on the OpenShift cluster; the client (browser UI) runs on the
user's machine. All commands entered in the client execute on the cluster —
so data being processed never leaves the cluster, and the cluster's larger
resources are available to the work.

A workbench can be configured with connections (external data access, model
save targets) and cluster storage (persistence). Workbenches in the same
project can share models/data with AI pipelines and model servers via object
storage.

Multiple workbenches per project are normal — split by configuration needs
(CPU/RAM/IDE) or by task/activity (e.g. one workbench per LLM experiment,
demo, or test).

Created via the procedure on [[creating-a-workbench]].

## Workbench images

A workbench image is preinstalled with the IDE plus tools/libraries for
model development; use a provided one or an admin-created custom image.
**As of 2.25**, the LLM Compressor library (model compression, especially
for vLLM deployment targets) is integrated into supported workbench images
and AI pipelines.

Provided images (RHOAI 3.5):

- **CUDA** — NVIDIA CUDA Toolkit, for GPU-accelerated libraries/tools.
- **Standard Data Science** — commonly-used ML libraries, no
  TensorFlow/PyTorch.
- **TensorFlow** — includes computational-graph visualization.
- **PyTorch** — for computer vision / NLP work.
- **Minimal Python** — no advanced ML features or extra compute resources.
- **TrustyAI** — model explainability, tracing, accountability, runtime
  monitoring.
- **code-server** — VS Code-style editor with extensions; note Elyra-based
  pipelines are unavailable in this image.
- **RStudio Server** (Technology Preview) — R IDE; unavailable in
  disconnected environments; must be built locally, see
  [[building-rstudio-workbench-images]]. Red Hat supports managing the
  workbench, not the RStudio software itself (separate licensing terms).
- **CUDA - RStudio Server** (Technology Preview) — RStudio IDE + CUDA
  Toolkit; same build/licensing caveats as RStudio Server.
- **ROCm** — AMD GPU support; used by [[enabling-amd-gpus]].
- **ROCm-PyTorch** — PyTorch on AMD GPUs.
- **ROCm-TensorFlow** — TensorFlow on AMD GPUs.

If the preinstalled packages in these images aren't sufficient: install
additional libraries ad hoc after launch (not saved across workbench
restarts — dependency management gets harder over time), or build a custom
image — see [[creating-custom-workbench-images]] (building it) and
[[importing-custom-workbench-images]] (making it selectable in the
dashboard). Custom images are also the only option for vendors without a
prebuilt image, e.g. Intel Gaudi — see [[enabling-intel-gaudi-accelerators]].
