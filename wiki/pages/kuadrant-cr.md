---
type: component
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/reference/kuadrant.md#L14-127
verified: 2026-08-26
confidence: documented
---

# Kuadrant CR

The `Kuadrant` custom resource is the top-level object that turns on the Kuadrant control plane in a namespace — it is the object that [[kuadrant-authpolicy]], [[kuadrant-ratelimitpolicy]], and [[kuadrant-tokenratelimitpolicy]] all depend on existing before those policies can be enforced. (raw/kuadrant-operator/doc/reference/kuadrant.md#L14-22)

A minimal instance looks like:

```yaml
apiVersion: kuadrant.io/v1beta1
kind: Kuadrant
metadata:
  name: kuadrant
  namespace: kuadrant-system
```

(raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L26-34 — this is the exact apply-block used to bootstrap Kuadrant before walking through TokenRateLimitPolicy)

## Namespace significance

The namespace the `Kuadrant` CR is deployed into matters beyond just where the CR object lives: [[kuadrant-authpolicy]] resources that reference API key `Secret`s (with the default `allNamespaces: false`) require those secrets to be created in the **Kuadrant CR's namespace**, not the AuthPolicy's own namespace. (raw/kuadrant-operator/doc/overviews/auth.md#L395-399)

## KuadrantSpec fields

`spec` is largely optional/blank; the fields that exist configure cross-cutting concerns for the whole deployment rather than any one policy: (raw/kuadrant-operator/doc/reference/kuadrant.md#L16-30)

| Field | Type | Description |
|---|---|---|
| `observability` | Observability | Telemetry/monitoring config for control- and data-plane components (logging, tracing). |
| `mtls` | mTLS | Turns on mutual-TLS between the gateway and Kuadrant's data-plane components (Authorino, Limitador). |
| `components` | Components | Optional component config, currently just `developerPortal.enabled`. |

(raw/kuadrant-operator/doc/reference/kuadrant.md#L23-30)

**mTLS**: `mtls.enable` is the master switch; `mtls.limitador` and `mtls.authorino` let you scope it to just one component. Per the documented truth table, a component-specific `true` only takes effect when `enable: true` is also set — setting a component flag alone (`enable: false, limitador: true`) does **not** turn on mTLS for that component. (raw/kuadrant-operator/doc/reference/kuadrant.md#L31-46)

**Observability**: `observability.dataPlane` controls OpenTelemetry trace-span filtering emitted by the WASM modules in the data plane (not `kubectl logs` verbosity), and `observability.tracing` configures where trace spans are exported (Jaeger/Zipkin/Tempo-compatible collector endpoint). (raw/kuadrant-operator/doc/reference/kuadrant.md#L61-107) — full observability/tracing configuration is out of scope for this ingest; noted here only as a field that exists on the CR.

**Components → developerPortal**: `components.developerPortal.enabled` turns on the `APIProduct` and `APIKeyRequest` CRDs for developer-portal integration; default `false`. (raw/kuadrant-operator/doc/reference/kuadrant.md#L108-119)

## Status

`status.mtlsLimitador` and `status.mtlsAuthorino` report the actual effective mTLS state per component (post truth-table resolution), alongside the standard `observedGeneration`/`conditions` pair. (raw/kuadrant-operator/doc/reference/kuadrant.md#L120-127)

## Related

- [[kuadrant-authpolicy]] — depends on this CR's namespace for API-key secrets.
- [[kuadrant-ratelimitpolicy]], [[kuadrant-tokenratelimitpolicy]] — data-plane policies enforced via the Kuadrant control plane this CR activates.
- [[kuadrant-secure-protect-connect-walkthrough]] — end-to-end procedure that assumes a `Kuadrant` CR is already installed.
- DNS (`DNSPolicy`), TLS (`TLSPolicy`), x509/mTLS client-cert auth, and observability/tracing are documented elsewhere upstream but out of scope for this ingest pass — see the walkthrough page for pointers.
