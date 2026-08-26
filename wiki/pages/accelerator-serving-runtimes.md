---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L238-327]
verified: 2026-08-25
confidence: documented
---

# Accelerator-specific serving runtimes

OpenShift AI supports specific accelerators through preinstalled [[kserve-model-serving-crds|ServingRuntimes]].
Each requires its own accelerator-enablement procedure on the cluster before the runtime can be
used.

| Accelerator | Preinstalled runtime | Cluster enablement (prerequisite) |
|---|---|---|
| NVIDIA GPU | vLLM NVIDIA GPU ServingRuntime for KServe | Node Feature Discovery Operator + [[enabling-nvidia-gpus]] |
| Intel Gaudi | vLLM Intel Gaudi Accelerator ServingRuntime for KServe | [[enabling-intel-gaudi-accelerators]] (Intel Gaudi Base Operator + hardware profile) |
| AMD GPU | vLLM AMD GPU ServingRuntime for KServe | [[enabling-amd-gpus]] (AMD GPU operator + hardware profile) |
| IBM Spyre (x86_64) | vLLM Spyre AI Accelerator ServingRuntime for KServe | Spyre Operator + hardware profile — Technology Preview on x86_64 in RHOAI 3.5 |
| IBM Spyre (s390x / IBM Z) | vLLM Spyre s390x ServingRuntime for KServe | Spyre Operator + hardware profile — GA since RHOAI 3.0 |
| IBM Spyre (ppc64le / IBM Power) | vLLM Spyre ppc64le ServingRuntime for KServe | Spyre Operator + hardware profile — GA since RHOAI 3.3 |

Note: Intel Gaudi runtime supports a "warm-up" step (model init / perf optimization that reduces
cold-start and first-inference latency, at the cost of longer load times). Skip it in
non-production environments by setting `VLLM_SKIP_WARMUP="true"` via
[[customizing-model-serving-runtime-parameters]].

## Supported vs. tested-and-verified runtimes

- **Preinstalled ("supported") runtimes** ship with OpenShift AI and work without modification —
  see Red Hat's Supported Configurations article for the current list (moving target, not
  reproduced here).
- **Tested-and-verified runtimes** are community model-serving runtimes Red Hat tests/verifies
  against a specific OpenShift AI version (retested each new OpenShift AI release; a runtime
  released mid-cycle is verified in an upcoming release). They are **not directly supported by Red
  Hat** — the customer is responsible for licensing and for configuring/maintaining them. Added via
  [[adding-tested-and-verified-runtime]].
- Either kind can be extended with your own **custom runtime** — [[adding-custom-model-serving-runtime]].

## See also

- [[model-serving-platforms-overview]]
- [[kserve-model-serving-crds]]
- [[enabling-model-serving-platform]]
- [[adding-custom-model-serving-runtime]]
- [[adding-tested-and-verified-runtime]]
