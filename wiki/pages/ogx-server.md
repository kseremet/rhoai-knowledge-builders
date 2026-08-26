---
type: component
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L922-1129
verified: 2026-08-25
confidence: documented
---

# OGXServer

`OGXServer` (`ogx.io/v1beta1`) is the custom resource for a RAG
(Retrieval-Augmented Generation) server instance managed by the OGX
Operator, RHOAI's integrated component for combining LLM inference,
semantic retrieval, and vector database storage. Enabled via
[[datasciencecluster]]'s `ogx` component. The cluster separately exposes an
`OGX` resource, an internal RHOAI resource managed by the [[rhoai-operator]]
— don't confuse it with `OGXServer`; the CR you configure directly is
`OGXServer`.

## Namespace

Namespaced — created per-project alongside the workload that uses it (e.g.
`my-ogx-namespace` in the examples below).

## Depends on

- [[dscinitialization]] and `odh-trusted-ca-bundle`, if using the
  recommended CA-bundle pattern (see [[configuring-ca-bundle-for-ogx-server]]).
- Node Feature Discovery Operator and NVIDIA GPU Operator (see
  [[gpu-operator-clusterpolicy]] family) if the server needs GPU-enabled
  nodes.
- Red Hat OpenShift Service Mesh Operator 3.x, cert-manager Operator (RAG
  workload requirements, per component-requirements list).
- S3-compatible object storage for model artifacts.

## The knob that matters: `spec.server.tlsConfig.caBundle`

By default the OGX server image trusts only public CAs. To let it
communicate over TLS with self-signed or private-CA inference/embedding/
vector-store providers, set `spec.server.tlsConfig.caBundle`:

- `configMapName` (required) — source config map with the CA cert(s).
- `configMapNamespace` (optional) — defaults to the `OGXServer` CR's own
  namespace; cross-namespace reads require the OGX Operator service
  account to have read access to that namespace.
- `configMapKeys` (optional) — list of keys in the source config map
  holding CA data; concatenated into one bundle. Defaults to just
  `ca-bundle.crt` if omitted. Set this when the source config map uses
  other key names — e.g. `odh-trusted-ca-bundle`, which holds data under
  both `ca-bundle.crt` and `odh-ca-bundle.crt`.

See [[configuring-ca-bundle-for-ogx-server]] for the runnable procedure and
[[ogx-server-ca-bundle-reference]] for validation rules, limits, and
failure conditions.

## What changing the CA bundle does

Changing/adding the CA bundle causes the OGX Operator to restart the OGX
server pod (brief service interruption) so new certificates take effect.
The Operator:

1. Reads and validates CA certs from the source config map.
2. Concatenates valid certs into a managed config map named
   `<instance-name>-ca-bundle`.
3. Mounts it into the OGX server pod at
   `/etc/ssl/certs/ca-bundle/ca-bundle.crt`.
4. Sets `SSL_CERT_FILE` on the server container to that path.

## Status

`status.conditions` publishes `DeploymentReady`, `ServiceReady`,
`HealthCheck` (each `"True"`/`"False"`, with a `message` describing
failures). `status.phase` summarizes overall state:
`Initializing` | `Ready` | `Failed`.
