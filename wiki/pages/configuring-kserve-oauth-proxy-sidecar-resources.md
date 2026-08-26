---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L393-421
verified: 2026-08-25
confidence: documented
---

# Configuring OAuth proxy sidecar resources for KServe

Customizes CPU/memory requests and limits for the OAuth proxy sidecar
container that KServe injects into inference service pods, to prevent
out-of-memory conditions when deploying large models. Edits the
[[datasciencecluster]] CR directly so the [[rhoai-operator]] keeps
reconciling it (avoids hand-editing the `inferenceservice-config`
ConfigMap, which the Operator would otherwise overwrite).

## Preconditions

- Cluster administrator access.
- The Operator is installed and a [[datasciencecluster]] CR exists with the
  `kserve` component in `Managed` state (see
  [[installing-rhoai-components]]).
- OpenShift CLI (`oc`) installed.

## Important

This rolls out **every** inference service that has authentication enabled
— plan for the resulting pod restarts.

## Steps

1. `$ oc edit DataScienceCluster default-dsc`
2. Under `spec.components.kserve`, add `oauthProxy.resources`:
   ```yaml
   spec:
     components:
       kserve:
         managementState: "Managed"
         oauthProxy:
           resources:
             requests:
               memory: "256Mi"
               cpu: "200m"
             limits:
               memory: "512Mi"
               cpu: "500m"
   ```
   Defaults if `oauthProxy` is omitted entirely (Operator defaults
   preserved): `requests.memory=64Mi`, `requests.cpu=100m`,
   `limits.memory=128Mi`, `limits.cpu=200m`. Setting only one sub-field
   (e.g. only `limits.memory`) overrides just that value; the rest keep
   their defaults. Values must be valid Kubernetes resource quantities
   (e.g. `64Mi`, `1Gi`, `100m`) — invalid strings are rejected by the API
   server.
3. Save and close the editor. The Operator applies the override to the
   `inferenceservice-config` ConfigMap and rolls out the
   `kserve-controller-manager` deployment; existing `InferenceService` pods
   roll out with the new sidecar sizing.

## Verification

1. `$ oc get configmap inferenceservice-config -n redhat-ods-applications -o jsonpath='{.data.oauthProxy}' | python3 -m json.tool`
   — shows the updated `memoryRequest`/`memoryLimit`/`cpuRequest`/`cpuLimit`.
2. `$ oc rollout status deployment/kserve-controller-manager -n redhat-ods-applications`
   — reports a successful rollout.
3. With at least one `InferenceService` running in the target namespace:
   `$ oc get pods -l serving.kserve.io/inferenceservice -n <your_namespace> -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{range .spec.containers[*]} {.name}: memory={.resources.limits.memory}{"\n"}{end}{end}'`
   — shows the `oauth-proxy` container's memory limit alongside
   `kserve-container`'s. Empty output if no `InferenceService` exists in
   that namespace yet.
