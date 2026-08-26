---
type: concept
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/overviews/auth.md#L14-399
verified: 2026-08-26
confidence: documented
---

# Kuadrant Auth (concept)

Why and when to use Kuadrant's [[kuadrant-authpolicy]] CRD, before getting into its field-level reference. Kuadrant auth secures Gateway API traffic (authentication + authorization) without requiring any change to the backend application. (raw/kuadrant-operator/doc/overviews/auth.md#L14-18)

## The jobs it solves

| Job | Solution |
|---|---|
| Secure API access with the right authentication method | API keys, JWT/OIDC, X.509 certs, OAuth2 introspection, Kubernetes TokenReview, or anonymous — chosen per use case |
| Verify cryptographic identity | X.509 client-cert auth with defense-in-depth at both TLS and application layers (out of scope for this ingest) |
| Fine-grained authorization | Pattern matching, OPA/Rego, Kubernetes RBAC (SubjectAccessReview), or SpiceDB |
| Consistent policy across many APIs | Apply at Gateway, HTTPRoute, or GRPCRoute level, with `defaults`/`overrides` to set baselines vs mandates |
| Centralize outbound-call credentials | Egress gateway credential injection (out of scope for this ingest) |

(raw/kuadrant-operator/doc/overviews/auth.md#L20-31)

## How it's enforced

Kuadrant uses the industry-standard Envoy External Authorization protocol: the gateway checks whether an [[kuadrant-authpolicy]] matches an incoming request, and if so calls out to **Authorino** (the external auth service), which evaluates the configured rules and returns `ALLOW`/`DENY`. This gives zero-app-change enforcement, consistent rules regardless of which service handles a request, defense-in-depth for methods like X.509 that validate at both L4/TLS and L7, and dynamic policy updates without redeploys. (raw/kuadrant-operator/doc/overviews/auth.md#L36-51)

## Choosing an authentication method

See [[kuadrant-authpolicy]] for the full one-of list and the decision table (API Key / JWT-OIDC / X.509 / OAuth2 Introspection / Kubernetes TokenReview / Plain / Anonymous). (raw/kuadrant-operator/doc/overviews/auth.md#L68-79)

## Choosing an authorization method

See [[kuadrant-authpolicy]] for Pattern Matching / OPA / Kubernetes SubjectAccessReview / SpiceDB. (raw/kuadrant-operator/doc/overviews/auth.md#L80-90)

## Governance model: defaults vs overrides

Route-targeted policies suit application teams owning their own endpoints; Gateway-targeted policies suit platform engineers setting org-wide baselines that automatically cover new routes. `defaults` establish a baseline a more specific policy can replace; `overrides` are mandatory and cannot be weakened by a more specific policy. This follows Gateway API [GEP-713](https://gateway-api.sigs.k8s.io/geps/gep-713/), and Kuadrant implements 4 merge strategies (atomic/merged × defaults/overrides). Without overrides, the most-specific-hostname-matching policy wins on wildcard overlaps. (raw/kuadrant-operator/doc/overviews/auth.md#L104-301)

`when` conditions let a policy activate on attributes HTTPRoute matching can't express — source IP ranges, JWT claims, time windows — using CEL predicates or JSON-path selectors against Kuadrant's Well-known Attributes. (raw/kuadrant-operator/doc/overviews/auth.md#L302-347)

## Egress use (forward reference, out of scope)

AuthPolicy can also authenticate internal workloads and inject externally-required credentials at an egress gateway, centralizing external API credential management/rotation instead of distributing secrets to every pod — documented in a separate egress-credential-injection guide not covered by this ingest. (raw/kuadrant-operator/doc/overviews/auth.md#L348-377)

## Known limitations

- Same-namespace targeting only.
- No mixed HTTPRoute/GRPCRoute on one Gateway hostname.
- API-key secrets (default `allNamespaces: false`) must live in the [[kuadrant-cr]]'s namespace.

(raw/kuadrant-operator/doc/overviews/auth.md#L395-399)

## Related

- [[kuadrant-authpolicy]] — the CRD this concept motivates; full field reference and worked toystore example.
- [[kuadrant-cr]] — must be installed first.
- [[kuadrant-rate-limiting-overview]], [[kuadrant-token-rate-limiting-overview]] — auth is frequently paired with rate limiting, using identity attributes AuthPolicy exposes.
- [[kuadrant-secure-protect-connect-walkthrough]] — runnable procedure applying these concepts.
- X.509 client-certificate auth, OIDC federation, and egress credential injection are documented upstream but out of scope for this ingest pass — see the linked overview doc for pointers if needed later.
