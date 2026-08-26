---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L492-577]
verified: 2026-08-25
confidence: documented
---

# Customizing the parameters of a deployed model-serving runtime

Modifies the runtime configuration (arguments/environment variables) for one specific model
deployment — a per-[[kserve-model-serving-crds|InferenceService]] customization that does not
change the default runtime configuration. Used for supporting specific models or enhancing an
existing deployment. Can be set at first deploy or by editing an existing deployment.

## Preconditions

- Logged in to the RHOAI dashboard as an OpenShift AI administrator.
- A model is already deployed.

## Steps

1. **AI hub → Deployments** in the dashboard.
2. Click **Stop** next to the deployment you want to customize.
3. Action menu (⋮) → **Edit**. The **Configuration parameters** section shows any predefined
   serving runtime parameters.
4. Edit:
   - **Additional serving runtime arguments** — defines how the deployed model behaves.
   - **Additional environment variables** — defines env vars in the model's environment.
   - Do not modify the port or model-serving-runtime arguments the dashboard sets — they need
     specific values; overwriting can break the deployment.
   - `VLLM_CPU_KVCACHE_SPACE` sets the vLLM KV cache size in GiB (e.g. `=40` for 40 GiB). Default
     `0` means vLLM allocates from system memory at runtime instead of reserving dedicated KV
     cache — which can cause out-of-memory errors; raise it to support more parallel requests,
     sized to your hardware/memory budget.
5. Click **Redeploy** to save.
6. Click **Start** to deploy with the changes.

## Verification

- The deployment shows a checkmark in the **Status** column on the project's **Deployments** tab
  and the dashboard's **Deployments** page.
- Confirm the arguments/env vars landed in `spec.predictor.model.args` /
  `spec.predictor.model.env`, either via the InferenceService YAML in the OpenShift console, or:
  `oc get -o json inferenceservice <inferenceservicename/modelname> -n <projectname>`

## Rollback

Repeat steps 2–3, remove the added arguments/variables, **Redeploy**, **Start**.

## Where to find parameter references

Per-runtime engine-argument documentation (external, not reproduced here — see the runtime
vendor's docs):

| Serving runtime | Where the parameters are documented |
|---|---|
| NVIDIA Triton Inference Server | NVIDIA Triton model-configuration docs |
| OpenVINO Model Server | OpenVINO dynamic-input-parameters docs |
| vLLM NVIDIA GPU / AMD GPU / Intel Gaudi ServingRuntime for KServe | vLLM engine-arguments (OpenAI-compatible server) docs |
| vLLM Spyre ppc64le ServingRuntime for KServe | Red Hat AI Inference Server docs, IBM Power + Spyre recommended settings |

## Worked examples: customizing the vLLM ServingRuntime for KServe by model family

Deploying a Llama, Granite, or Mistral model may need additional flags beyond the defaults.
Preconditions specific to these examples: the **vLLM ServingRuntime for KServe** runtime is
enabled; the relevant model is downloaded to object storage (meta-llama-3;
granite-7b-instruct/granite-20B-code-instruct; or mistral-7B-Instruct-v0.3); GPU support and the
Node Feature Discovery Operator are enabled ([[enabling-nvidia-gpus]]).

During deployment, select **vLLM ServingRuntime for KServe** as the serving runtime, then add
under **Additional serving runtime arguments**:

- **meta-llama-3**: `--distributed-executor-backend=mp --max-model-len=6144` (multiprocessing
  backend for distributed workers; 6144-token max context).
- **granite-7B-instruct**: `--distributed-executor-backend=mp`.
- **granite-20B-code-instruct**: `--distributed-executor-backend=mp --tensor-parallel-size=4
  --max-model-len=6448` (distributes inference across 4 GPUs in a single node; 6448-token max
  context).
- **mistral-7B-Instruct-v0.3**: `--distributed-executor-backend=mp --max-model-len=15344`.

Click **Deploy**.

Verification (granite models): confirm the checkmark on **Models**/**Model deployments**, then:
```
curl -q -X 'POST' "https://<inference_endpoint_url>:443/v1/chat/completions" \
  -H 'accept: application/json' -H 'Content-Type: application/json' \
  -d "{ \"model\": \"<model_name>\", \"prompt\": \"<prompt>\", \"max_tokens\": <max_tokens>, \"temperature\": <temperature> }"
```

## See also

- [[kserve-model-serving-crds]]
- [[accelerator-serving-runtimes]]
- [[configuring-speculative-decoding-and-multimodal-inferencing]]
- [[setting-default-model-deployment-strategy]]
- [[enabling-nvidia-gpus]]
