---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L410-446]
verified: 2026-08-25
confidence: documented
---

# Adding a tested-and-verified model-serving runtime

Adds and enables a Red Hat tested-and-verified [[kserve-model-serving-crds|ServingRuntime]] (see
[[accelerator-serving-runtimes]] for what "tested and verified" means) via the RHOAI dashboard.
These are not directly supported by Red Hat.

## Preconditions

- Logged in to the RHOAI dashboard as an OpenShift AI administrator.
- Deploying the IBM Z Accelerated for NVIDIA Triton Inference Server runtime: access to the IBM
  Cloud Container Registry to pull the image.
- Deploying the IBM Power Accelerated Triton Inference Server runtime: access to the Triton
  Inference Server Quay repository.

## Steps

1. **Settings → Model resources and operations → Serving runtimes** — shows installed/enabled
   runtimes.
2. Click **Add serving runtime**.
3. Select **Single-model serving platform** as the supported platform.
4. Select **REST** or **gRPC** as the API protocol.
5. Click **Start from scratch**.
6. Paste the ServingRuntime YAML for the runtime you're adding. The guide gives four worked
   examples (`metadata.name` values below are the ones shown; pick a name that doesn't collide
   with a runtime you've already added):
   - **IBM Power Accelerated for NVIDIA Triton Inference Server** (REST) — `name:
     triton-ppc64le-runtime`, image `quay.io/powercloud/tritonserver:latest`, formats FIL/python/
     onnx/pytorch, container port 8000.
   - **IBM Z Accelerated for NVIDIA Triton Inference Server** — REST variant `name:
     ibmz-triton-rest` or gRPC variant `name: ibmz-triton-grpc`, image
     `icr.io/ibmz/ibmz-accelerated-for-nvidia-triton-inference-server:<version>`, formats
     onnx-mlir/snapml/pytorch, `securityContext` hardened (drop ALL caps, `runAsNonRoot: true`,
     `RuntimeDefault` seccomp), protocol versions v2/grpc-v2. The gRPC variant mounts a `shm`
     `emptyDir` volume (2Gi).
   - **NVIDIA Triton Inference Server** — REST variant `name: triton-kserve-rest` or gRPC variant
     `name: triton-kserve-grpc`, image `nvcr.io/nvidia/tritonserver@sha256:<digest>` (a moving
     value — use the digest current at deploy time), Prometheus annotations for metrics scraping,
     formats tensorrt/tensorflow (v1 and v2)/onnx/pytorch/triton/xgboost/python.
   - Full YAML for each is in the raw source (see this page's `sources`).
7. In `metadata.name`, confirm the value doesn't collide with a runtime already added.
8. Optional: set a custom display name via `metadata.annotations['openshift.io/display-name']`
   (otherwise the dashboard shows `metadata.name`).
9. Click **Create**. The runtime is added and auto-enabled; the chosen API protocol is shown.
10. Optional: edit later via action menu (⋮) → **Edit**.

## Verification

The added runtime appears in an enabled state on the **Serving runtimes** page.

## Rollback

Not documented — disable or remove the runtime via its action menu (⋮).

## See also

- [[kserve-model-serving-crds]]
- [[accelerator-serving-runtimes]]
- [[adding-custom-model-serving-runtime]]
- [[enabling-model-serving-platform]]
