---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L33-77]
verified: 2026-08-25
confidence: documented
---

# Model-serving platforms overview

RHOAI's model-serving workflow: upload a trained model to S3-compatible object storage, a
persistent volume claim, or an OCI image; access and train it from a project workbench
([[rhoai-workbench]]); then serve/deploy it on a model-serving platform. Deploying makes the
model available as a service (a model runtime server) with an inference endpoint you can query
via API ("model inferencing") — visible from the RHOAI dashboard.

As of RHOAI 3.5, an administrator chooses between two serving platforms per business need:

## Model serving platform

Deploys each model on its own dedicated runtime server. Built on [KServe](https://github.com/kserve/kserve).
Recommended for production; suited to large models such as LLMs and generative AI. Configured
through [[kserve-model-serving-crds]] (the `ServingRuntime` and `InferenceService` CRDs), and
requires the `kserve` component to be enabled via [[datasciencecluster]] — see
[[installing-rhoai-components]].

## NVIDIA NIM model serving platform

Deploys NVIDIA-optimized models using NVIDIA Inference Microservices (NIM), part of NVIDIA AI
Enterprise — a set of microservices for GPU-accelerated inferencing (virtual screening, content
generation, avatar creation, etc.). The NIM platform is **built on top of** the model serving
platform: the model serving platform must be installed and enabled first. See
[[enabling-nvidia-nim-model-serving-platform]].

## Choosing

- Want each model on its own runtime server (LLMs, gen AI, production use)? → model serving
  platform.
- Want to deploy NVIDIA-optimized models as NIMs? → NVIDIA NIM model serving platform (which
  itself depends on the model serving platform being enabled).

## See also

- [[kserve-model-serving-crds]] — the CRDs the model serving platform uses
- [[accelerator-serving-runtimes]] — which preinstalled runtimes support which accelerators
- [[enabling-model-serving-platform]] — the enablement procedure
- [[enabling-nvidia-nim-model-serving-platform]] — the NIM-specific enablement procedure
- [[red-hat-openshift-ai]] — general RHOAI workflow context
