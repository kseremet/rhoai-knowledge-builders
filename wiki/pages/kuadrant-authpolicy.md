---
type: component
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/reference/authpolicy.md#L14-282
  - raw/kuadrant-operator/doc/overviews/auth.md#L14-399
  - raw/kuadrant-operator/examples/toystore/authpolicy.yaml.md#L1-65
  - raw/kuadrant-operator/examples/toystore/admin-key-secret.yaml.md#L1-30
  - raw/kuadrant-operator/examples/toystore/alice-api-key-secret.yaml.md#L1-29
  - raw/kuadrant-operator/examples/toystore/bob-api-key-secret.yaml.md#L1-29
verified: 2026-08-26
confidence: documented
---

# AuthPolicy (Kuadrant)

`AuthPolicy` is a Kuadrant custom resource (`kuadrant.io/v1`) that attaches authentication and authorization rules to Gateway API traffic. It requires the [[kuadrant-cr]] to be installed. (raw/kuadrant-operator/doc/overviews/auth.md#L14-18)

## What it does

It targets a `Gateway`, `HTTPRoute`, or `GRPCRoute` (optionally a specific `sectionName` — a listener or route rule) and, when matched traffic arrives, has the gateway call out to **Authorino** (the external auth service) over the Envoy External Authorization (ext_authz) protocol. Authorino evaluates the configured authentication/authorization rules and returns `ALLOW` or `DENY`. (raw/kuadrant-operator/doc/overviews/auth.md#L28-50, raw/kuadrant-operator/doc/reference/authpolicy.md#L23-34)

Because auth logic lives in the policy rather than application code, this requires zero changes to the backend service. (raw/kuadrant-operator/doc/overviews/auth.md#L44-50)

## Spec shape

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: my-auth-policy
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute   # or GRPCRoute, Gateway
    name: myroute
  when: [...]
  patterns: {...}
  rules:
    authentication: {...}
    metadata: {...}
    authorization: {...}
    response: {...}
    callbacks: {...}
  defaults: {...}     # mutually exclusive with top-level rules/when/patterns
  overrides: {...}    # mutually exclusive with defaults and top-level rules/when/patterns
```

(raw/kuadrant-operator/doc/reference/authpolicy.md#L160-282 — full high-level example)

Top-level `spec.rules`/`spec.when`/`spec.patterns` are shorthand for implicit defaults, and are mutually exclusive with an explicit `spec.defaults` block; `spec.overrides` is mutually exclusive with both. (raw/kuadrant-operator/doc/reference/authpolicy.md#L23-34)

### AuthScheme (`spec.rules` / `defaults.rules` / `overrides.rules`)

| Field | Purpose |
|---|---|
| `authentication` | Map of named authentication rules. At least one must resolve to a valid identity, or the auth request fails. Empty/omitted = anonymous access. |
| `metadata` | Rules for fetching auth metadata from external sources (HTTP, OIDC UserInfo, UMA). |
| `authorization` | Map of named authorization rules. **All** must allow for the request to succeed. |
| `response` | Customizes the auth response — headers/denial status for unauthenticated/unauthorized/success cases. |
| `callbacks` | Post-authorization webhook calls, fired regardless of the auth decision. |

(raw/kuadrant-operator/doc/reference/authpolicy.md#L43-48)

Every rule in each of these maps also inherits `when` (rule-scoped activation conditions), `cache`, `priority` (rules in the same priority group evaluate concurrently; different groups evaluate sequentially), and `metrics`. (raw/kuadrant-operator/doc/reference/authpolicy.md#L53-60)

### Authentication methods (`AuthenticationRule`, one-of)

`apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous` — pick exactly one per named rule. (raw/kuadrant-operator/doc/reference/authpolicy.md#L62-77)

| Method | When to use |
|---|---|
| API Key | Service-to-service, developer access, simple token auth |
| JWT / OIDC | End-user auth, SSO, federated identity |
| X.509 client cert | Zero-trust, cryptographic identity, mTLS compliance — see the (out-of-scope for this ingest) X.509 overview doc |
| OAuth2 Introspection | Validating tokens from an external OAuth2/OIDC server |
| Kubernetes TokenReview | In-cluster ServiceAccount-token auth |
| Plain | Identity pre-resolved by an upstream proxy |
| Anonymous | Public/unauthenticated routes |

(raw/kuadrant-operator/doc/overviews/auth.md#L68-79)

### Authorization methods (`AuthorizationRule`, one-of)

`patternMatching`, `opa`, `kubernetesSubjectAccessReview`, `spicedb`. (raw/kuadrant-operator/doc/reference/authpolicy.md#L87-96)

| Method | When to use |
|---|---|
| Pattern Matching | Simple CEL-expression attribute checks |
| OPA (Rego) | Complex business-rule authorization, policy-as-code |
| Kubernetes SubjectAccessReview | Reuse existing RBAC Roles/RoleBindings |
| SpiceDB | Fine-grained, relationship-based (Zanzibar-style) permissions |

(raw/kuadrant-operator/doc/overviews/auth.md#L80-90)

### Response customization

`spec.rules.response.success.headers` / `.filters` let you inject values (plain text, JSON, or a Festival Wristband token) into the request headers or into Kuadrant's internal filter chain — `filters` is how identity attributes extracted here (e.g. `userid`, `groups`) get made available to [[kuadrant-ratelimitpolicy]] and [[kuadrant-tokenratelimitpolicy]] for use in `counters`/`when`. (raw/kuadrant-operator/doc/reference/authpolicy.md#L97-120)

## Targeting and policy hierarchy

- **Route-targeted** (`HTTPRoute`/`GRPCRoute`, optionally `sectionName` for one rule): for application teams protecting their own endpoints. (raw/kuadrant-operator/doc/overviews/auth.md#L108-155)
- **Gateway-targeted**: for platform engineers setting org-wide baselines; automatically covers routes attached later. (raw/kuadrant-operator/doc/overviews/auth.md#L192-216)
- **`defaults` vs `overrides`** (per Gateway API [GEP-713](https://gateway-api.sigs.k8s.io/geps/gep-713/)): `defaults` establish a baseline a more specific (route-level) policy can replace; `overrides` win unconditionally over more specific policies and cannot be weakened. Policies with neither block behave as defaults. Kuadrant implements 4 merge strategies: atomic defaults, atomic overrides, merged defaults, merged overrides. (raw/kuadrant-operator/doc/overviews/auth.md#L217-262)
- **Wildcard hostnames**: without overrides, "most specific hostname wins" — an AuthPolicy on `a.toystore.com` beats one on `*.toystore.com` for requests to `a.toystore.com`. (raw/kuadrant-operator/doc/overviews/auth.md#L282-301)
- **`when` conditions**: activate a policy/rule based on request attributes HTTPRoute matching can't express (source IP, JWT claims, time). Support CEL predicates or JSON-path selector modifiers against Kuadrant's [Well-known Attributes](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md). (raw/kuadrant-operator/doc/overviews/auth.md#L302-347)

## Known limitations

- AuthPolicy can only target resources in its own namespace. (raw/kuadrant-operator/doc/overviews/auth.md#L395-397)
- A Gateway cannot mix HTTPRoute and GRPCRoute on the same hostname — Gateway API enforces hostname uniqueness across route types. (raw/kuadrant-operator/doc/overviews/auth.md#L397-398)
- API-key `Secret`s referenced with `allNamespaces: false` (the default) must live in the [[kuadrant-cr]]'s namespace, not the AuthPolicy's namespace. (raw/kuadrant-operator/doc/overviews/auth.md#L398-399)

## Worked example: toystore API-key auth

`examples/toystore/authpolicy.yaml.md` demonstrates two `AuthPolicy` resources stacked at different levels: one on the `toystore` `HTTPRoute` requiring an API key labeled `app: toystore` and exposing the resolved `user-id` annotation as `ext_auth_data`; and a second, in `gateway-system`, on the `kuadrant-ingressgateway` `Gateway` itself, requiring an API key additionally labeled `admin: "yes"` — i.e. a stricter, admin-only auth rule guarding the whole gateway. (raw/kuadrant-operator/examples/toystore/authpolicy.yaml.md#L1-65)

The three companion secrets show how `apiKey` selectors and the `secret.kuadrant.io/user-id` annotation pattern work in practice:
- `toystore-admin-apikey` — labeled `app: toystore, admin: "yes"`, annotated `user-id: alice`, key `IAMADMIN`. (raw/kuadrant-operator/examples/toystore/admin-key-secret.yaml.md#L1-30)
- `toystore-alice-apikey` — labeled `app: toystore`, annotated `user-id: alice`, key `ALICEKEYFORDEMO`. (raw/kuadrant-operator/examples/toystore/alice-api-key-secret.yaml.md#L1-29)
- `toystore-bob-apikey` — labeled `app: toystore`, annotated `user-id: bob`, key `BOBKEYFORDEMO`. (raw/kuadrant-operator/examples/toystore/bob-api-key-secret.yaml.md#L1-29)

All three carry `authorino.kuadrant.io/managed-by: authorino`, which is what makes Authorino pick them up as API-key credential sources.

## Egress use (forward reference)

AuthPolicy attaches to egress Gateways/HTTPRoutes identically to ingress, and can be used to authenticate an internal workload before injecting externally-required credentials at the gateway (centralized credential management/rotation). This is documented in a separate egress-credential-injection user guide, out of scope for this ingest. (raw/kuadrant-operator/doc/overviews/auth.md#L348-377)

## Related

- [[kuadrant-cr]] — must exist first; also governs where API-key secrets live.
- [[kuadrant-ratelimitpolicy]], [[kuadrant-tokenratelimitpolicy]] — commonly stacked with AuthPolicy, consuming identity attributes AuthPolicy exposes via `response.success.filters`.
- [[kuadrant-auth-overview]] — concept-level rationale for choosing an authentication/authorization method.
- [[kuadrant-secure-protect-connect-walkthrough]], [[kuadrant-authenticated-token-rate-limiting-tutorial]] — procedures that configure AuthPolicy end to end.
- X.509 client-certificate auth, OIDC federation, and egress credential injection are documented upstream but out of scope for this ingest pass.
