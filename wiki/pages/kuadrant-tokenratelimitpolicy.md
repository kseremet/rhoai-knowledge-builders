---
type: component
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L14-236
  - raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-246
  - raw/kuadrant-operator/examples/toystore/tokenratelimitpolicy_httproute.yaml.md#L1-33
verified: 2026-08-26
confidence: documented
---

# TokenRateLimitPolicy (Kuadrant)

`TokenRateLimitPolicy` is a Kuadrant custom resource (`kuadrant.io/v1alpha1`) that rate-limits AI/LLM API traffic by **actual token consumption** rather than request count. It requires the [[kuadrant-cr]] to be installed. This is the CRD directly relevant to LLM/inference token-cost billing and quota enforcement. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

## What it does

It targets a `Gateway` or `HTTPRoute`. It automatically extracts token usage from OpenAI-compatible API responses — specifically the `/usage/total_tokens` JSON pointer — and sends that count to Limitador as the `hits_addend` on the Envoy RLS `RateLimitRequest`, so the counter increments by actual tokens used, not by 1 per request. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23, raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L114-136)

Request flow: on a matching request, the gateway evaluates `when`/`counters`; on the response, it extracts `usage.total_tokens` from the body; it sends the RLS request with that value as `hits_addend`; Limitador tracks cumulative token usage and responds `OK`/`OVER_LIMIT`. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

**Graceful fallback**: if token parsing fails, the policy falls back to counting the request itself. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L69-78)

## Supported response formats

Works out-of-the-box, zero config, with any backend returning `usage.total_tokens` in an OpenAI-compatible shape:

```json
{"usage": {"total_tokens": 150, "prompt_tokens": 100, "completion_tokens": 50}}
```

(raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L114-133)

Confirmed-supported endpoints/backends: OpenAI Chat Completions (`/v1/chat/completions`), OpenAI legacy Completions (`/v1/completions`), OpenAI Embeddings, and OpenAI-compatible backends — **vLLM, kServe, Ollama, Azure OpenAI, and Gemini's OpenAI-compatibility endpoint**. (raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L133-136, raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L69-78) — the kServe/vLLM support is the direct link to RHOAI model-serving: see [[model-serving-platforms-overview]] and [[kserve-model-serving-crds]].

**Not supported yet**: providers returning usage under a different field name — Anthropic `/v1/messages` (`usage.input_tokens`/`output_tokens`, no `total_tokens`) and Google Gemini's native endpoint (`usageMetadata.totalTokenCount`). Tracked upstream in [Kuadrant/kuadrant-operator#1864](https://github.com/Kuadrant/kuadrant-operator/issues/1864). (raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L136, raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

**Streaming**: non-streaming works with `stream: false`/omitted. Streaming requires the request to set `"stream": true` **and** `"stream_options": {"include_usage": true}` — otherwise usage cannot be extracted from the final stream event, and depending on the policy's `failureMode` the request is either allowed unlimited or rejected. (raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L106-136, raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L353-376)

## Spec shape

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: llm-protection
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway   # or HTTPRoute
    name: ai-gateway
  limits:
    "<name>":
      rates: [{limit: <token-count>, window: <duration>}]
      when: [{predicate: "..."}]
      counters: [{expression: "..."}]
  defaults:
    strategy: atomic   # or merge
    limits: {...}
  overrides:            # Gateway-targeted only
    strategy: atomic
    limits: {...}
```

(raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L23-84)

Note the field is `rates[].limit` = a **token count**, not a request count, and `window` follows the same Gateway API Duration format as [[kuadrant-ratelimitpolicy]]. (raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L65-71)

## Policy hierarchy: three modes

1. **Implicit defaults** — `spec.limits` set directly: applied to the target; if targeting a Gateway, a more specific route policy can override it. Most common for single-policy scenarios. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L115-120)
2. **Explicit defaults** — `spec.defaults`: baseline rules for routes lacking a more specific policy; supports `strategy: atomic|merge`; can be overridden by route-level policies or Gateway `overrides`. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L121-127)
3. **Overrides** — `spec.overrides`: wins over everything else in the hierarchy; **Gateway-targeted only**; used for org-wide caps that can't be bypassed. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L128-134)

## Integration with AuthPolicy

TokenRateLimitPolicy has no authentication of its own — user-tier segmentation (free/gold/etc.) depends on [[kuadrant-authpolicy]] exposing identity attributes via `response.success.filters`, which then become available as `auth.identity.*` in `TokenRateLimitPolicy`'s `when`/`counters` CEL expressions. The reference doc gives a worked AuthPolicy fragment that authenticates by API key, groups users via an OPA rule checking `kuadrant.io/groups`, and filters `groups`/`userid` into the identity object for TokenRateLimitPolicy to key off of. (raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md — see kuadrant-operator/doc/overviews/token-rate-limiting.md#L195-239 for the fragment)

## Example patterns from the reference doc

User segmentation by tier (free/gold), gated on both path and a groups-claim CEL predicate, counted per user:

```yaml
limits:
  free-tier:
    rates: [{limit: 20000, window: 24h}]
    when: [{predicate: 'auth.identity.groups.split(",").exists(g, g == "free")'}]
    counters: [{expression: auth.identity.userid}]
```

(raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L79-94)

Multiple time windows on one limit, for burst + sustained + daily protection simultaneously:

```yaml
limits:
  burst-protection:
    rates:
    - {limit: 1000, window: 1m}      # burst
    - {limit: 50000, window: 1h}     # sustained
    - {limit: 500000, window: 1d}    # daily quota
```

(raw/kuadrant-operator/doc/reference/tokenratelimitpolicy.md#L95-110)

Org-wide override that cannot be bypassed by any team-level policy:

```yaml
spec:
  overrides:
    limits:
      org-total:
        rates: [{limit: 10000000, window: 720h}]
        counters: [{expression: auth.identity.org_id}]
```

(raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L171-194)

## Worked example: toystore free-tier token limit

`examples/toystore/tokenratelimitpolicy_httproute.yaml.md` targets the `toystore` HTTPRoute with a single `free` limit (20,000 tokens/day), gated on `request.auth.claims["kuadrant.io/groups"]` containing `"free"`, counted per `auth.identity.userid`. Note this uses `request.auth.claims[...]` rather than `auth.identity.metadata.annotations[...]` to read the groups claim — a JWT-claims-style access pattern distinct from the API-key-annotation pattern used elsewhere in the toystore examples. (raw/kuadrant-operator/examples/toystore/tokenratelimitpolicy_httproute.yaml.md#L1-33)

## Related

- [[kuadrant-cr]] — must exist first.
- [[kuadrant-authpolicy]] — supplies the identity/group attributes this policy's `when`/`counters` key off of.
- [[kuadrant-ratelimitpolicy]] — the request-count sibling CRD; use that instead when token cost isn't the concern.
- [[kuadrant-token-rate-limiting-overview]] — concept-level rationale for token-based vs request-based limiting.
- [[kuadrant-authenticated-token-rate-limiting-tutorial]] — full runnable procedure standing this up against a simulated LLM service.
- [[model-serving-platforms-overview]], [[kserve-model-serving-crds]] — kServe/vLLM inference endpoints are explicitly named as compatible backends; this is the concrete mechanism for token-cost-based rate limiting in front of an RHOAI-served model, though no RHOAI-specific integration doc exists yet (that's a later ingest's job per this ingest's scope note).
