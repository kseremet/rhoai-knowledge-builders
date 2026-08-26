---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L350-378]
verified: 2026-08-25
confidence: documented
---

# Configuring speculative decoding and multi-modal inferencing (vLLM NVIDIA GPU runtime)

Configures the **vLLM NVIDIA GPU ServingRuntime for KServe** (see
[[accelerator-serving-runtimes]]) for:
- **speculative decoding** — a parallel-processing technique to optimize LLM inferencing time;
- **multi-modal inferencing** — support for vision-language models (VLMs), a subset of
  multi-modal models combining visual and textual data.

## Preconditions

- Logged in to the RHOAI dashboard as an OpenShift AI administrator.
- For speculative decoding with a draft model: the original model and the speculative model are
  stored in the same folder in S3-compatible object storage.

## Steps

1. Deploy a model, selecting the **vLLM NVIDIA GPU ServingRuntime for KServe** runtime in the
   **Serving runtime** field (deployment flow itself is out of scope for this guide — see the
   dashboard's model-deployment flow).
2. Speculative decoding by n-gram matching: under **Additional serving runtime arguments** in
   **Configuration parameters**, add:
   `--speculative-model=[ngram] --num-speculative-tokens=<NUM_SPECULATIVE_TOKENS> --ngram-prompt-lookup-max=<NGRAM_PROMPT_LOOKUP_MAX> --use-v2-block-manager`
   (replace the two placeholders). Note: throughput varies by the model used for n-gram
   speculation.
3. Speculative decoding with a draft model: add
   `--port=8080 --served-model-name={{.Name}} --distributed-executor-backend=mp --model=/mnt/models/<path_to_original_model> --speculative-model=/mnt/models/<path_to_speculative_model> --num-speculative-tokens=<NUM_SPECULATIVE_TOKENS> --use-v2-block-manager`
   (replace the model-path and token-count placeholders).
4. Multi-modal inferencing: add `--trust-remote-code`. Only use this argument with models from
   trusted sources.
5. Click **Deploy**.

## Verification

- Speculative decoding:
  `curl -v https://<inference_endpoint_url>:443/v1/chat/completions -H "Content-Type: application/json" -H "Authorization: Bearer <token>"`
- Multi-modal inferencing (VLM):
  `curl -v https://<inference_endpoint_url>:443/v1/chat/completions -H "Content-Type: application/json" -H "Authorization: Bearer <token>" -d '{"model":"<model_name>", "messages": [{"role":"<role>", "content": [{"type":"text","text":"<text>"},{"type":"image_url","image_url":"<image_url_link>"}]}]}'`

## Rollback

Not documented — remove the added arguments from **Configuration parameters** and redeploy (same
mechanism as [[customizing-model-serving-runtime-parameters]]).

## See also

- [[kserve-model-serving-crds]]
- [[accelerator-serving-runtimes]]
- [[customizing-model-serving-runtime-parameters]]
