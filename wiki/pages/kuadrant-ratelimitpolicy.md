---
type: component
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/reference/ratelimitpolicy.md#L14-158
  - raw/kuadrant-operator/doc/overviews/rate-limiting.md#L14-327
  - raw/kuadrant-operator/examples/toystore/ratelimitpolicy_gateway.yaml.md#L1-39
  - raw/kuadrant-operator/examples/toystore/ratelimitpolicy_httproute.yaml.md#L1-56
verified: 2026-08-26
confidence: documented
---

# RateLimitPolicy (Kuadrant)

`RateLimitPolicy` is a Kuadrant custom resource (`kuadrant.io/v1`) that attaches request-count-based rate limits to Gateway API traffic. It requires the [[kuadrant-cr]] to be installed. For token-count-based limiting (the LLM-relevant variant), see [[kuadrant-tokenratelimitpolicy]] instead. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L23-34)

## What it does

It targets a `Gateway`, `HTTPRoute`, or `GRPCRoute` (optionally scoped to a `sectionName`). On a matching request, the gateway sends an Envoy [Rate Limit Service (RLS)](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto) `RateLimitRequest` to **Limitador** (the external rate-limiting service), which responds `OK` or `OVER_LIMIT`. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L23-34)

## Spec shape

```yaml
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: my-rate-limit-policy
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute   # or GRPCRoute, Gateway
    name: myroute
  limits:
    "my_limit":
      rates: [...]      # required — the actual rate limits
      counters: [...]   # optional — per-value counters
      when: [...]       # optional — activation conditions
  defaults:
    limits: {...}       # mutually exclusive with top-level `limits`
  overrides:
    limits: {...}       # mutually exclusive with `defaults` and top-level `limits`; Gateway-targeted only
```

(raw/kuadrant-operator/doc/reference/ratelimitpolicy.md#L112-158)

`spec.limits` is a map of named limit definitions and is mutually exclusive with `spec.defaults`. `spec.overrides` is only valid when `targetRef.kind: Gateway`. (raw/kuadrant-operator/doc/reference/ratelimitpolicy.md#L24-35)

### Limit definition fields

| Field | Required | Description |
|---|---|---|
| `rates[]` (`RateLimit`) | for enforcement | `{limit: <number>, window: <Gateway API Duration>}` — e.g. `{limit: 100, window: 1s}`. |
| `counters[]` (`Counter`) | No | CEL expressions (`expression`) resolved per-request; each distinct resolved value gets its own counter, keyed against each `rates` entry. |
| `when[]` (`Predicate`) | No | CEL predicates; **all** must evaluate true for the limit to activate. |

(raw/kuadrant-operator/doc/reference/ratelimitpolicy.md#L54-87)

A limit activates when the request matches all its `when` predicates; if `counters` is set, a separate counter is tracked per distinct resolved counter value (e.g. per username); if omitted, the limit is a single global counter. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L160-182)

Example — three limits on one policy, showing rates/counters/when together:

```yaml
spec:
  limits:
    "toystore-all":
      rates: [{limit: 5000, window: 1s}]
    "toystore-api-per-username":
      rates: [{limit: 100, window: 1s}, {limit: 1000, window: 1m}]
      counters: [{expression: auth.identity.username}]
      when: [{predicate: "request.host == 'api.toystore.com'"}]
    "toystore-admin-unverified-users":
      rates: [{limit: 250, window: 1s}]
      when:
        - {predicate: "request.host == 'admin.toystore.com'"}
        - {predicate: "!auth.identity.email_verified"}
```

(raw/kuadrant-operator/doc/overviews/rate-limiting.md#L173-207)

Note the `counters` expression (`auth.identity.username`) reads an attribute [[kuadrant-authpolicy]] exposes via `response.success.filters`.

## Targeting and policy hierarchy

- **Route-targeted**: enforced on all traffic matched by the route's rules/hostnames, across every Gateway in the route's `parentRefs`. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L65-77)
- **Gateway-targeted `defaults`** (or neither `defaults`/`overrides` specified): covers all traffic through the gateway except where a more specific HTTPRoute/GRPCRoute RateLimitPolicy exists — new routes are automatically covered. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L94-106)
- **Gateway-targeted `overrides`**: enforced on **all** routes through the gateway regardless of any more specific route-level policy — atomically replaces it. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L122-159)
- **Hostname specificity**: without overrides, most-specific-hostname-wins — policies aren't merged, only one applies per request. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L78-84)

## Known limitations

- RateLimitPolicy can only target resources in its own namespace. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L227-230)
- Mixed HTTPRoute/GRPCRoute on the same Gateway hostname isn't supported (Gateway API hostname-uniqueness rule). (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L230-231)

## Implementation notes (data plane)

Because Istio only supports one filter chain per listener and Gateway API HTTPRoute rules have no stable "name" to key Envoy's native Rate Limit filter descriptors on, Kuadrant does **not** use Envoy's Rate Limit HTTP filter directly. Instead it generates a `WasmPlugin` (Istio [Wasm Network filter](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/wasm_filter)) — the [wasm-shim](https://github.com/Kuadrant/wasm-shim) — that implements the RLS protocol against Limitador, letting multiple rate-limit "domains"/scopes coexist per gateway with fine-grained per-route-rule matching. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L232-260) This `WasmPlugin` is auto-generated by the Kuadrant control plane from your `RateLimitPolicy` + targeted Gateway API resource — you never author it directly. (raw/kuadrant-operator/doc/overviews/rate-limiting.md#L295-327)

## Worked example: toystore rate limits

`examples/toystore/ratelimitpolicy_gateway.yaml.md` — a Gateway-targeted policy on `kuadrant-ingressgateway` with two limits: `expensive-operation` (2 requests/30s, gated on `request.method == 'POST'`) and `limit-per-ip` (5 requests/30s, gated on `source.id == source.address`). Because it declares `limits` at the top level (not `defaults`/`overrides`), it behaves as a default that a route-level policy can supersede. (raw/kuadrant-operator/examples/toystore/ratelimitpolicy_gateway.yaml.md#L1-39)

`examples/toystore/ratelimitpolicy_httproute.yaml.md` — a route-targeted policy on the `toystore` HTTPRoute with four limits: a `global` catch-all (6 req/30s), `get-toy` scoped to `GET /toy` (5 req/1m), and two per-admin-user limits (`admin-post-toy-per-user`, `admin-delete-per-user`, each 2 req/30s) counted by `auth.identity.username` and gated on `auth.identity.group == 'admin'` plus method/path predicates. (raw/kuadrant-operator/examples/toystore/ratelimitpolicy_httproute.yaml.md#L1-56)

## Related

- [[kuadrant-cr]] — must exist first.
- [[kuadrant-authpolicy]] — commonly stacked underneath/above; supplies identity attributes used in `counters`/`when`.
- [[kuadrant-tokenratelimitpolicy]] — the token-count sibling of this CRD, for LLM/AI token-based limits instead of request counts.
- [[kuadrant-rate-limiting-overview]] — concept-level rationale and targeting patterns.
- [[kuadrant-secure-protect-connect-walkthrough]] — procedure that configures both a Gateway-level default and an HTTPRoute-level override RateLimitPolicy.
- Rate-limiting an inference endpoint by request count (rather than token count) could apply to [[model-serving-platforms-overview]] deployments fronted by a Gateway — no direct integration is documented in this slice, noted only as a natural connection point.
