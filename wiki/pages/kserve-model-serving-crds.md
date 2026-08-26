---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L84-237]
verified: 2026-08-25
confidence: documented
---

# ServingRuntime and InferenceService (KServe CRDs)

The two CRDs (`serving.kserve.io`) that together configure a model server on the
[[model-serving-platforms-overview|model serving platform]] (KServe-based, single-model serving).
Deps: requires the `kserve` [[datasciencecluster]] component enabled — see
[[installing-rhoai-components]] — and, once enabled, an OAuth proxy sidecar sized per
[[configuring-kserve-oauth-proxy-sidecar-resources]].

## ServingRuntime

Creates the serving runtime itself: an environment for deploying/managing a model. Generates the
pod templates that dynamically load/unload models of a given format, and exposes a service
endpoint for inferencing requests.

Key `spec` fields (from the example vLLM ServingRuntime for KServe):
- `metadata.annotations['opendatahub.io/recommended-accelerators']` — recommended accelerator(s)
  for the runtime, e.g. `nvidia.com/gpu`.
- `metadata.annotations['openshift.io/display-name']` — display name shown in the dashboard.
- `spec.annotations['prometheus.io/path']` / `['prometheus.io/port']` — Prometheus scrape
  endpoint/port for the runtime's metrics.
- `spec.containers[].args` — includes the model path (e.g. `--model=/mnt/models`) and
  `--served-model-name={{.Name}}`, a template variable that maps to the `InferenceService`'s
  `spec.predictor.name`.
- `spec.containers[].image` — the runtime container image; differs by accelerator type (a moving
  value — check the raw source/cluster for the current pinned tag, not this page).
- `spec.multiModel: false` — marks the runtime as single-model.
- `spec.supportedModelFormats` — model formats the runtime accepts (e.g. `vLLM`).

See [[accelerator-serving-runtimes]] for the preinstalled, accelerator-specific ServingRuntimes
RHOAI ships, and [[adding-custom-model-serving-runtime]] / [[adding-tested-and-verified-runtime]]
for adding your own.

## InferenceService

Creates the server/inference service that receives inference queries, passes them to the model,
and returns the output. It:
- specifies the model's location and format,
- specifies which ServingRuntime serves the model,
- can enable the passthrough route for gRPC or REST inference,
- defines the HTTP/gRPC endpoints for the deployed model.

Key `spec.predictor` fields (from the example, deploying a granite model with the vLLM runtime):
- `model.modelFormat.name` — must match a format the chosen ServingRuntime supports.
- `model.runtime` — the ServingRuntime's display name.
- `model.storage.key` / `model.storage.path` — the object-storage connection and path to the
  model files.
- `model.resources` / `tolerations` — CPU/memory/accelerator requests-limits and any accelerator
  toleration (e.g. `nvidia.com/gpu`); treat exact sizings as moving values, not fixed defaults.
- `minReplicas` / `maxReplicas` — replica bounds for the predictor.

Verify a deployment's runtime parameters with:
```
oc get -o json inferenceservice <inferenceservicename> -n <projectname>
```
(used in [[customizing-model-serving-runtime-parameters]]'s verification step).

## See also

- [[model-serving-platforms-overview]]
- [[accelerator-serving-runtimes]]
- [[enabling-model-serving-platform]]
- [[customizing-model-serving-runtime-parameters]]
- [[adding-custom-model-serving-runtime]]
- [[adding-tested-and-verified-runtime]]
