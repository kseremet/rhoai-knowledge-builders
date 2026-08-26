---
type: concept
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-246
verified: 2026-08-26
confidence: documented
---

# Kuadrant Token Rate Limiting (concept)

Why and when to use [[kuadrant-tokenratelimitpolicy]] instead of the plain request-count [[kuadrant-ratelimitpolicy]]. This is the CRD in this ingest slice most directly relevant to LLM/AI token-cost billing, since traditional per-request rate limiting doesn't reflect the wildly varying compute cost of different LLM calls. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

## What it does differently

Instead of counting requests, `TokenRateLimitPolicy` automatically extracts `usage.total_tokens` from OpenAI-compatible response bodies and sends that count to Limitador as `hits_addend` on the RLS request — so a 10-token completion and a 2,000-token completion consume very different amounts of quota, even though both are "one request." It also integrates with [[kuadrant-authpolicy]] for user/tier-based limiting using authentication claims. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

**Scope of automatic extraction**: any backend returning `usage.total_tokens` in an OpenAI-compatible shape — OpenAI itself, and OpenAI-compatible backends including **vLLM and kServe** (the RHOAI-relevant ones), Ollama, Azure OpenAI, Gemini's OpenAI-compat endpoint. Anthropic and Gemini's native formats are not yet parsed (tracked in [Kuadrant/kuadrant-operator#1864](https://github.com/Kuadrant/kuadrant-operator/issues/1864)). Only `/v1/chat/completions` and `/v1/completions`-shaped responses are supported today. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

Streaming responses are supported but require the request to set `"stream": true` and `"stream_options": {"include_usage": true}`, or usage can't be extracted from the final stream chunk. (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L14-23)

## Why this matters for RHOAI / MaaS

This slice is upstream Kuadrant only (`products: [kuadrant]`) — it does not itself document a RHOAI or Models-as-a-Service integration. But because kServe/vLLM inference endpoints are named as directly-compatible backends, TokenRateLimitPolicy is the mechanism a later `products: [maas]`/`products: [rhoai]` ingest would build a token-cost-billing or per-tenant-quota page on top of. See [[model-serving-platforms-overview]] and [[kserve-model-serving-crds]] for the RHOAI-side serving components this would front.

## Segmentation and multi-window patterns

Different limits per user tier (free/gold/etc.), gated on a groups claim CEL predicate and counted per user ID, is the standard shape:

```yaml
limits:
  free-tier:
    rates: [{limit: 20000, window: 24h}]
    when: [{predicate: 'auth.identity.groups.split(",").exists(g, g == "free")'}]
    counters: [{expression: auth.identity.userid}]
```

Multiple `rates` entries on one limit protect against both burst and sustained overconsumption simultaneously (e.g. 1k tokens/min AND 50k/hour AND 500k/day, all on one `limits` entry). (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L79-110)

## Policy hierarchy: three modes

**Implicit defaults** (top-level `limits`, most common for single-policy setups), **explicit defaults** (`spec.defaults`, baseline for routes lacking a more specific policy, supports `atomic`/`merge` strategies), and **overrides** (`spec.overrides`, Gateway-targeted only, wins unconditionally — for org-wide quotas that can't be bypassed). (raw/kuadrant-operator/doc/overviews/token-rate-limiting.md#L111-134)

## Related

- [[kuadrant-tokenratelimitpolicy]] — the CRD this concept motivates; full field reference and the toystore worked example.
- [[kuadrant-ratelimitpolicy]] — the request-count sibling; use when token cost isn't the concern.
- [[kuadrant-authpolicy]], [[kuadrant-auth-overview]] — user/tier segmentation depends on identity attributes AuthPolicy exposes.
- [[kuadrant-authenticated-token-rate-limiting-tutorial]] — runnable procedure standing up token rate limiting end to end against a simulated LLM service.
- [[model-serving-platforms-overview]], [[kserve-model-serving-crds]] — the RHOAI-side serving components this policy type is designed to sit in front of; no RHOAI-specific integration page exists yet.
