=== UNIT ID: kuadrant-operator/doc/overviews/auth.md:common-authentication-and-authorization-jobs ===
DOC: kuadrant-operator/doc/overviews/auth.md
TITLE: Common authentication and authorization jobs
BREADCRUMB: ['Kuadrant Auth', 'Secure API access with authentication and authorization']
TOKENS: 1874
## Secure API access with authentication and authorization

When protecting APIs in your Kubernetes infrastructure, you need to verify who is making requests (authentication) and what they're allowed to do (authorization). Kuadrant provides flexible authentication and authorization capabilities through the AuthPolicy custom resource, helping you balance security requirements with operational simplicity.

Kuadrant AuthPolicy helps you accomplish these common security tasks:

| Job | When you need this | Solution |
|-----|-------------------|----------|
| **Secure API access with appropriate authentication** | APIs need protection but different use cases require different authentication methods | Choose from API keys, JWT/OIDC, X.509 certificates, OAuth2 introspection, Kubernetes TokenReview, or allow anonymous access based on your security and compliance requirements |
| **Verify cryptographic identity with client certificates** | Your organization requires strong identity verification, zero-trust architecture, or compliance with security standards | Use X.509 client certificate authentication with defense-in-depth validation at both TLS and application layers |
| **Control access with fine-grained authorization** | Authenticated users need different permissions based on roles, attributes, or complex business logic | Enforce authorization rules using pattern matching, OPA policies, Kubernetes RBAC, or SpiceDB |
| **Apply consistent security policies across API infrastructure** | You manage multiple APIs and routes and need to balance developer flexibility with platform governance | Apply AuthPolicies at Gateway, HTTPRoute, or GRPCRoute level with defaults and overrides to establish security baselines |
| **Secure outbound API calls with centralized credential management** | Services call external APIs requiring authentication, and you want to avoid distributing secrets to every pod | Use egress gateway credential injection to authenticate internal workloads and inject external API credentials at the gateway |

## Understanding how Kuadrant auth integrates with your infrastructure

When evaluating Kuadrant for API security, you'll want to understand how it fits into your existing Kubernetes and Gateway API infrastructure without requiring changes to your application code.

## How authentication requests are processed

Kuadrant uses the industry-standard [Envoy External Authorization](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/ext_authz_filter) protocol to enforce authentication and authorization policies:

1. **Request arrives**: Your Gateway (Istio, Envoy Gateway, etc.) receives an API request
2. **Policy evaluation**: The gateway checks if any AuthPolicy matches the request based on the HTTPRoute or Gateway configuration
3. **External authorization call**: If a policy matches, the gateway sends an authorization check request to Authorino (the external auth service)
4. **Authentication and authorization**: Authorino evaluates your authentication rules (API key, JWT, X.509, etc.) and authorization policies
5. **Decision returned**: Authorino responds with either `ALLOW` (request proceeds) or `DENY` (request rejected with appropriate status code)

This approach provides several benefits:
- **Zero application changes**: Authentication logic lives in policies, not in your code
- **Consistent enforcement**: Same auth rules apply regardless of which service handles the request
- **Defense in depth**: Some authentication methods (like X.509) validate at both the gateway (L4/TLS) and application (L7) layers
- **Dynamic policy updates**: Change authentication requirements without redeploying services

## What AuthPolicy provides

A Kuadrant AuthPolicy custom resource:

1. **Targets Gateway API resources**: Attaches to [HTTPRoute](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.HTTPRoute), [GRPCRoute](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.GRPCRoute), or [Gateway](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.Gateway) to define which traffic to protect
2. **Supports fine-grained targeting**: Apply policies to specific Gateway listeners or individual HTTPRoute/GRPCRoute rules using `sectionName`
3. **Abstracts complexity**: Simplifies Envoy External Authorization configuration while providing access to powerful authentication methods
4. **Enables platform governance**: Platform engineers can set `defaults` to establish security baselines that apply until application teams define more specific policies
5. **Enforces mandatory controls**: Platform engineers can set `overrides` to enforce organization-wide security requirements that cannot be weakened by route-level policies

Check out the [API reference](../reference/authpolicy.md) for the complete AuthPolicy CRD specification.

## Choosing the right authentication method

When securing API access, different use cases require different authentication approaches. Choose the authentication method that best matches your security requirements, infrastructure, and user experience needs.

## Authentication method decision guide

| Authentication Method | When to use | Key characteristics | Learn more |
|-----------------------|-------------|---------------------|------------|
| **API Key** | Service-to-service communication, developer access, simple token-based auth, small-to-mid scale deployments | Simple to implement; secrets stored in Kubernetes; supports namespace-scoped or cluster-wide keys | [API Key authentication](https://docs.kuadrant.io/latest/authorino/docs/features/#api-key-authenticationapikey) |
| **JWT / OIDC** | End-user authentication, SSO, federated identity | Industry-standard tokens; integrates with identity providers (Keycloak, Auth0, Okta); supports token validation and claims extraction | [JWT verification](https://docs.kuadrant.io/latest/authorino/docs/features/#jwt-verification-authenticationjwt) |
| **X.509 Client Certificates** | Zero-trust architecture, cryptographic identity, compliance requirements (mTLS), machine-to-machine with PKI | Strongest authentication; defense-in-depth validation; multi-CA trust with label selectors; requires certificate infrastructure | [X.509 authentication](auth-x509.md) |
| **OAuth2 Introspection** | Validate tokens issued by external OAuth2/OIDC servers | Delegates validation to token issuer; works with opaque tokens; real-time revocation check | [OAuth2 introspection](https://docs.kuadrant.io/latest/authorino/docs/features/#oauth-20-introspection-authenticationoauth2introspection) |
| **Kubernetes TokenReview** | In-cluster service-to-service auth, ServiceAccount tokens | Native Kubernetes integration; validates ServiceAccount tokens; leverages existing RBAC | [Kubernetes TokenReview](https://docs.kuadrant.io/latest/authorino/docs/features/#kubernetes-tokenreview-authenticationkubernetestokenreview) |
| **Plain Identity** | Pre-authenticated requests, custom authentication handled by upstream proxy | Identity extracted from context; authentication performed elsewhere; minimal overhead | [Plain authentication](https://docs.kuadrant.io/latest/authorino/docs/features/#plain-authenticationplain) |
| **Anonymous Access** | Public endpoints, unauthenticated access to specific routes | No authentication required; useful with conditional policies or selective route protection | [Anonymous access](../user-guides/auth/anonymous-access.md) |

=== UNIT ID: kuadrant-operator/doc/overviews/auth.md:choosing-an-authorization-method ===
DOC: kuadrant-operator/doc/overviews/auth.md
TITLE: Choosing an authorization method
BREADCRUMB: ['Kuadrant Auth', 'Choosing the right authentication method']
TOKENS: 1482
After authentication, you may need to enforce authorization rules to control what authenticated users can access:

| Authorization Method | When to use | Key characteristics | Learn more |
|---------------------|-------------|---------------------|------------|
| **Pattern Matching** | Simple attribute-based rules, checking claims or metadata values | CEL expressions; fast evaluation; good for straightforward conditions | [Pattern-matching authorization](https://docs.kuadrant.io/latest/authorino/docs/features/#pattern-matching-authorization-authorizationpatternmatching) |
| **OPA (Open Policy Agent)** | Complex authorization logic, business rules, policy-as-code | Rego policy language; centralized policy management; supports sophisticated evaluation | [OPA authorization](https://docs.kuadrant.io/latest/authorino/docs/features/#open-policy-agent-opa-rego-policies-authorizationopa) |
| **Kubernetes SubjectAccessReview** | Leverage Kubernetes RBAC, in-cluster authorization | Native K8s integration; uses existing Roles and RoleBindings; consistent with cluster permissions | [Kubernetes SubjectAccessReview](https://docs.kuadrant.io/latest/authorino/docs/features/#kubernetes-subjectaccessreview-authorizationkubernetessubjectaccessreview) |
| **SpiceDB** | Fine-grained permissions, relationship-based access control | External Authzed/SpiceDB server; scales to complex permission models; Google Zanzibar-inspired | [SpiceDB authorization](https://docs.kuadrant.io/latest/authorino/docs/features/#spicedb-authorizationspicedb) |

## X.509 client certificate authentication

For cryptographic identity verification using X.509 client certificates with mTLS, see the dedicated [X.509 Client Certificate Authentication overview](auth-x509.md). This authentication method is ideal for zero-trust architectures, compliance requirements, and machine-to-machine communication with PKI.

The X.509 overview covers:
- Configuration tiers (Gateway API v1.5+, provider-specific resources, header-only)
- Defense-in-depth security model with L4 and L7 validation
- Multi-CA trust with label selectors
- Certificate requirements and constraints
- Security considerations and best practices
- Troubleshooting guide

## Apply consistent security policies across your API infrastructure

When managing multiple APIs, routes, and gateways, you need flexible policy targeting that balances developer autonomy with platform governance. AuthPolicy provides multiple targeting options to match your organizational structure and security requirements.

## Protect specific routes with HTTPRoute or GRPCRoute-targeted policies

**Use route-targeted policies when:** Application teams own their routes and need to define authentication/authorization appropriate for their specific API endpoints.

An AuthPolicy targeting an HTTPRoute or GRPCRoute can protect:
- **All traffic** for the entire route, or
- **Specific rules** within the route by using `sectionName` to target individual route rules

The policy applies to all hostnames and gateways referenced by the HTTPRoute or GRPCRoute. Use top-level `when` conditions (`spec.rules.when`) for additional filtering based on request attributes.

**Example - Protect an entire HTTPRoute:**

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: my-auth
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: my-route
  rules: { … }
```

```
┌─────────────────────┐            ┌─────────────────────┐
│ (Gateway namespace) │            │   (App namespace)   │
│                     │            │                     │
│    ┌─────────┐      │ parentRefs │  ┌────────────┐     │
│    │ Gateway │◄─────┼────────────┼──┤ HTTPRoute  │     │
│    └─────────┘      │            │  | (my-route) │     |
│                     │            │  └────────────┘     │
│                     │            │        ▲            │
│                     │            │        │            │
│                     │            │        │ targetRef: │
│                     │            │        │ - my-route │
│                     │            │        │            │
│                     │            │  ┌─────┴──────┐     │
│                     │            │  │ AuthPolicy │     │
│                     │            │  │ (my-auth)  │     │
│                     │            │  └────────────┘     │
└─────────────────────┘            └─────────────────────┘
```

**Example - Protect only specific rules within a HTTPRoute:**

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: my-route-auth
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: my-route
    sectionName: rule-2
  rules: { … }
```

```
┌─────────────────────┐            ┌──────────────────────┐
│ (Gateway namespace) │            │    (App namespace)   │
│                     │            │                      │
│    ┌─────────┐      │ parentRefs │  ┌────────────┐      │
│    │ Gateway │◄─────┼────────────┼──┤ HTTPRoute  │      │
│    └─────────┘      │            │  | (my-route) │      |
│                     │            │  |------------│      |
│                     │            │  | - rule-1   │      |
│                     │            │  | - rule-2   │      |
│                     │            │  └────────────┘      │
│                     │            │        ▲             │
│                     │            │        │             │
│                     │            │        │ targetRef:  │
│                     │            │        │ - rule-2    │
│                     │            │        │             │
│                     │            │  ┌─────┴───────────┐ │
│                     │            │  │   AuthPolicy    │ │
│                     │            │  │ (my-route-auth) │ │
│                     │            │  └─────────────────┘ │
└─────────────────────┘            └──────────────────────┘
```

=== UNIT ID: kuadrant-operator/doc/overviews/auth.md:establish-platform-wide-security-baselines-with-gateway-targ ===
DOC: kuadrant-operator/doc/overviews/auth.md
TITLE: Establish platform-wide security baselines with Gateway-targeted policies
BREADCRUMB: ['Kuadrant Auth', 'Apply consistent security policies across your API infrastructure']
TOKENS: 1647
**Use Gateway-targeted policies when:** Platform engineers need to enforce organization-wide security requirements, establish default authentication for all routes, or ensure no route can be deployed without minimum security controls.

An AuthPolicy targeting a Gateway automatically applies to all routes attached to that gateway, including routes created after the policy. More specific HTTPRoute or GRPCRoute-targeted policies can override gateway defaults (unless you use `overrides` instead of `defaults`).

Key behaviors:
- **Automatic coverage**: New HTTPRoutes or GRPCRoutes attached to the gateway are automatically protected
- **Defaults allow specificity**: Route-level policies override gateway defaults, enabling application teams to customize authentication
- **Overrides enforce mandates**: Gateway overrides cannot be weakened by route-level policies, ensuring compliance requirements

**Example - Establish default authentication for all routes:**

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: my-gw-auth
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: my-gw
  defaults: # alternatively: `overrides`
    rules: { … }
```

```
┌───────────────────┐             ┌──────────────────────┐
│ (Infra namespace) │             │    (App namespace)   │
│                   │             │                      │
│  ┌─────────┐      │  parentRefs │  ┌───────────┐       │
│  │ Gateway │◄─────┼─────────────┼──┤ HTTPRoute │       │
│  | (my-gw) |      │             │  | (my-route)│       │
│  └─────────┘      │             │  └───────────┘       │
│       ▲           │             │        ▲             │
│       │           │             │        │             │
│       │ targetRef:│             │        │ targetRef:  │
│       │ - my-gw   │             │        │ - my-route  │
│       │           │             │        │             │
│ ┌─────┴────────┐  │             │  ┌─────┴───────────┐ │
│ │  AuthPolicy  │  │             │  │   AuthPolicy    │ │
│ | (my-gw-auth) |  │             │  │ (my-route-auth) │ │
│ └──────────────┘  │             │  └─────────────────┘ │
└───────────────────┘             └──────────────────────┘
```

## Balance platform governance with application flexibility using Defaults and Overrides

When managing security across multiple teams and services, you need to balance two goals: establishing organization-wide security baselines while giving application teams the flexibility to customize authentication for their specific needs.

AuthPolicy provides two mechanisms based on Gateway API [GEP-713](https://gateway-api.sigs.k8s.io/geps/gep-713/):

| Use | When | Example scenario |
|-----|------|------------------|
| **Defaults** (`spec.defaults`) | You want to establish security baselines that application teams can customize or override | Platform team sets API key authentication at the Gateway; app teams can override with JWT for specific routes |
| **Overrides** (`spec.overrides`) | You need to enforce mandatory security requirements that cannot be weakened | Security compliance requires all production traffic to use mTLS; override ensures no route can bypass this requirement |

**How defaults work:**
- Policies without explicit `defaults` or `overrides` blocks behave as defaults
- Default rules apply until a more specific policy supersedes them
- Application teams can replace defaults with route-specific policies
- Enables a "secure by default, customize as needed" approach

**Example use case for defaults:** Set a "deny-all" policy at the Gateway to protect against unintentional exposure. Application teams then define specific authentication rules for their routes, effectively opening controlled access.

**How overrides work:**
- Override rules win over any more specific policies
- Enforces mandatory controls that cannot be weakened by route-level policies
- Ensures organization-wide compliance requirements are always met

**Example use case for overrides:** Enforce mutual TLS for all production traffic regardless of what application teams configure, ensuring compliance with security standards.

**Policy hierarchy:**

Defaults and overrides work at any level of the Gateway API hierarchy:
1. Gateway (broadest)
2. Gateway listener (via `sectionName`)
3. HTTPRoute / GRPCRoute
4. HTTPRouteRule / GRPCRouteRule (via `sectionName`) (most specific)

The "effective policy" for each request is computed based on hierarchy rules from [GEP-713](https://gateway-api.sigs.k8s.io/geps/gep-713/#resolving-conflicts). Kuadrant implements 4 [merge strategies](https://gateway-api.sigs.k8s.io/geps/gep-713/#designing-a-merge-strategy) allowing atomic or merged composition of policy rules:
- **Atomic defaults:** The most specific policy fully replaces less specific ones for the topological scope it applies to. No merging of rules occurs. This is the default behavior.
- **Atomic overrides:** The broadest policy fully replaces more specific ones. No merging of rules occurs.
- **Merged defaults:** Individual rules from the most specific policy replace rules with the same name in less specific ones, while rules with different names are both maintained.
- **Merged overrides:** Individual rules from the broadest policy replace rules with the same name in more specific ones, while rules with different names are both maintained.

> [!NOTE] Learn more about Defaults & Overrides
> For detailed behavior and examples of Defaults & Overrides used in the AuthPolicies, see [RFC-0009](https://github.com/Kuadrant/architecture/blob/main/rfcs/0009-defaults-and-overrides.md).

## Understand how policies apply to wildcard hostnames

When you define routes with wildcard hostnames (e.g., `*.example.com`) alongside specific hostnames (e.g., `api.example.com`), you may wonder which AuthPolicy takes precedence.

**Principle: "Most specific wins"**

Without overrides in place, the AuthPolicy targeting the most specific matching route applies. Policies don't combine—only one policy's rules are enforced per request.

**Example scenario:**
- AuthPolicy A → HTTPRoute A (`a.toystore.com`)
- AuthPolicy B → HTTPRoute B (`b.toystore.com`)
- AuthPolicy W → HTTPRoute W (`*.toystore.com`)

**Request behavior:**
- Request to `a.toystore.com` → AuthPolicy A applies (specific hostname wins)
- Request to `b.toystore.com` → AuthPolicy B applies (specific hostname wins)
- Request to `other.toystore.com` → AuthPolicy W applies (wildcard catches unmatched)

This allows you to set broad defaults with wildcards while overriding authentication for specific subdomains.

=== UNIT ID: kuadrant-operator/doc/overviews/auth.md:apply-policies-conditionally-based-on-request-attributes-wit ===
DOC: kuadrant-operator/doc/overviews/auth.md
TITLE: Apply policies conditionally based on request attributes with `when` conditions
BREADCRUMB: ['Kuadrant Auth', 'Apply consistent security policies across your API infrastructure']
TOKENS: 1612
**Use `when` conditions when:** You need to activate authentication based on request attributes that can't be expressed in HTTPRoute matching rules, such as source IP ranges, identity-based attributes, resource attributes, custom metadata, or time-based conditions.

Common use cases:
- Apply different authentication to requests from internal vs. external IP ranges
- Enable authentication only during specific time windows
- Apply policies based on JWT claims or other dynamic attributes

`when` conditions support complex boolean expressions with AND/OR operators and grouping, compatible with Authorino [conditions](https://docs.kuadrant.io/latest/authorino/docs/features/#common-feature-conditions-when).

**Example - Apply different authentication based on source IP (internal vs. external traffic):**

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: network-based-auth
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: my-api
  rules:
    authentication:
      # Allow anonymous access from internal networks
      "internal-anonymous":
        anonymous: {}
        when:
        - predicate: "source.address.matches('10\\..*|172\\.(1[6-9]|2[0-9]|3[01])\\..*|192\\.168\\..*')"

      # Require strong authentication from external networks
      "external-mtls":
        x509:
          selector:
            matchLabels:
              tier: production
        when:
        # Activate when source is NOT in private IP ranges
        - predicate: "!source.address.matches('10\\..*|172\\.(1[6-9]|2[0-9]|3[01])\\..*|192\\.168\\..*')"
```

This example shows a scenario that cannot be achieved with HTTPRoute matching alone—HTTPRoute has no support for source IP-based routing or matching.

Conditions implement Kuadrant's [Well-known Attributes](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md). You can use [Common Expression Language (CEL)](https://cel.dev/) predicates or [JSON path selector modifiers](https://docs.kuadrant.io/latest/authorino/docs/features/#string-modifiers) for advanced conditions.

## Secure outbound API calls with credential injection at egress gateways

When your services call external APIs that require authentication, you need to centrally manage external credentials at the egress gateway rather than distributing secrets to every pod. This approach maintains security, simplifies credential rotation, and provides centralized audit logging for external API access.

## How egress credential injection works

**Use egress credential injection when:**
- Services need to call external APIs (payment processors, cloud providers, SaaS platforms)
- External APIs require authentication (API keys, OAuth tokens, custom headers)
- You want to avoid distributing external API credentials to every pod
- You need centralized credential management and rotation
- You want to authenticate internal workloads before allowing external API access

**The credential injection pattern:**

1. **Authenticate internal workload**: Gateway authenticates the workload making the outbound request using internal mechanisms (Kubernetes ServiceAccount tokens, JWT, API keys)
2. **Fetch external credentials**: Gateway retrieves external API credentials from secure storage (Kubernetes Secrets, Vault, external metadata sources)
3. **Inject credentials**: Gateway replaces or adds authentication headers in the outbound request with external API credentials
4. **Forward request**: Request proceeds to the external API with proper authentication


**Key characteristics:**
- **Same attachment model**: AuthPolicy attaches to egress Gateways and HTTPRoutes identically to ingress—Kuadrant doesn't distinguish between ingress and egress
- **Centralized secret management**: External credentials stored in one location, not distributed to pods
- **Credential rotation**: Update credentials at the gateway without redeploying workloads
- **Audit trail**: All external API access logged at the gateway
- **Defense in depth**: Authenticate internal workloads before allowing external API access

See the [Egress Gateway Credential Injection](../user-guides/egress/credential-injection.md) user guide for a complete walkthrough including HTTPRoute configuration for external services.

## Examples and user guides

Explore these user guides for complete examples of protecting services with Kuadrant AuthPolicy:

## Authentication examples

- [X.509 Client Certificate Authentication](../user-guides/auth/x509-authentication.md) - Defense-in-depth mTLS with three tier options (Gateway API v1.5+, provider-specific, or header-only)
- [Anonymous Access](../user-guides/auth/anonymous-access.md) - Selectively allow unauthenticated access to specific routes
- [Enforcing authentication & authorization with Kuadrant AuthPolicy, for app developers and platform engineers](../user-guides/auth/auth-for-app-devs-and-platform-engineers.md) - API key authentication with platform governance

## Egress gateway examples

- [Egress Gateway Credential Injection](../user-guides/egress/credential-injection.md) - Centralized credential management for outbound API calls

## Authentication with rate limiting

- [Authenticated Rate Limiting for Application Developers](../user-guides/ratelimiting/authenticated-rl-for-app-developers.md)
- [Authenticated Rate Limiting with JWTs and Kubernetes RBAC](../user-guides/ratelimiting/authenticated-rl-with-jwt-and-k8s-authnz.md)
- [Authenticated Rate Limiting for gRPC Services](../user-guides/ratelimiting/authenticated-rl-for-app-developers-grpc-services.md)

## Known limitations

- AuthPolicies can only target HTTPRoutes/GRPCRoutes/Gateways defined within the same namespace of the AuthPolicy.
- Mixed HTTPRoute and GRPCRoute sharing the same hostname on a Gateway is not supported. Gateway implementations enforce [hostname uniqueness](https://gateway-api.sigs.k8s.io/concepts/api-overview/#hostname-matching) across route types — use separate hostnames for HTTP and gRPC traffic.
- AuthPolicies that reference other Kubernetes objects (typically `Secret`s) require those objects to the created in the same namespace as the `Kuadrant` custom resource managing the deployment. This is the case of AuthPolicies that define API key authentication with `allNamespaces` option set to `false` (default), where the API key Secrets must be created in the Kuadrant CR namespace and not in the AuthPolicy namespace.

=== UNIT ID: kuadrant-operator/doc/overviews/rate-limiting.md:envoys-rate-limit-service-protocol ===
DOC: kuadrant-operator/doc/overviews/rate-limiting.md
TITLE: Envoy's Rate Limit Service Protocol
BREADCRUMB: ['Kuadrant Rate Limiting', 'How it works']
TOKENS: 1216
## Kuadrant Rate Limiting

A Kuadrant RateLimitPolicy custom resource, often abbreviated "RateLimitPolicy":

1. Targets Gateway API networking resources such as [HTTPRoutes](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.HTTPRoute), [GRPCRoutes](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.GRPCRoute), and [Gateways](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.Gateway), using these resources to obtain additional context, i.e., which traffic workload (HTTP attributes, hostnames, user attributes, etc) to rate limit.
2. Supports targeting subsets (sections) of a network resource to apply the limits to.
3. Abstracts the details of the underlying Rate Limit protocol and configuration resources, that have a much broader remit and surface area.
4. Enables cluster operators to set defaults that govern behavior at the lower levels of the network, until a more specific policy is applied.

Kuadrant's Rate Limit implementation relies on the Envoy's [Rate Limit Service (RLS)](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto) protocol. The workflow per request goes:

1. On incoming request, the gateway checks the matching rules for enforcing rate limits, as stated in the RateLimitPolicy custom resources and targeted Gateway API networking objects
2. If the request matches, the gateway sends one [RateLimitRequest](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto#service-ratelimit-v3-ratelimitrequest) to the external rate limiting service ("Limitador").
3. The external rate limiting service responds with a [RateLimitResponse](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto#service-ratelimit-v3-ratelimitresponse) back to the gateway with either an `OK` or `OVER_LIMIT` response code.

A RateLimitPolicy and its targeted Gateway API networking resource contain all the statements to configure both the ingress gateway and the external rate limiting service.

## The RateLimitPolicy custom resource

#### Overview

The `RateLimitPolicy` spec includes, basically, two parts:

- A reference to an existing Gateway API resource (`spec.targetRef`)
- Limit definitions (`spec.limits`)

Each limit definition includes:

- A set of rate limits (`spec.limits.<limit-name>.rates[]`)
- (Optional) A set of dynamic counter qualifiers (`spec.limits.<limit-name>.counters[]`)
- (Optional) A set of additional dynamic conditions to activate the limit (`spec.limits.<limit-name>.when[]`)

The limit definitions (`limits`) can be declared at the top-level level of the spec (with the semantics of _defaults_) or alternatively within explicit `defaults` or `overrides` blocks.

<table>
  <tbody>
    <tr>
      <td>Check out Kuadrant <a href="https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md">RFC 0002</a> to learn more about the Well-known Attributes that can be used to define counter qualifiers (<code>counters</code>) and conditions (<code>when</code>).</td>
    </tr>
  </tbody>
</table>

Check out the [API reference](../reference/ratelimitpolicy.md) for a full specification of the RateLimitPolicy CRD.

## Targeting a HTTPRoute or GRPCRoute networking resource

When a RateLimitPolicy targets a HTTPRoute or GRPCRoute, the policy is enforced to all traffic routed according to the rules and hostnames specified in the route, across all Gateways referenced in the `spec.parentRefs` field of the route.

Target a HTTPRoute (or GRPCRoute) by setting the `spec.targetRef` field of the RateLimitPolicy as follows:

```yaml
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: <RateLimitPolicy name>
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: <HTTPRoute Name>
  limits: { … }
```

![Rate limit policy targeting a HTTPRoute resource](../images/rlp-httproute-target.png)

#### Hostnames and wildcards

If a RateLimitPolicy targets a route defined for `*.com` and another RateLimitPolicy targets another route for `api.com`, the Kuadrant control plane will not merge these two RateLimitPolicies. Unless one of the policies declare an _overrides_ set of limites, the control plane will configure to mimic the behavior of gateway implementation by which the "most specific hostname wins", thus enforcing only the corresponding applicable policies and limit definitions.

E.g., by default, a request coming for `api.com` will be rate limited according to the rules from the RateLimitPolicy that targets the route for `api.com`; while a request for `other.com` will be rate limited with the rules from the RateLimitPolicy targeting the route for `*.com`.

See more examples in [Overlapping Gateway and HTTPRoute RateLimitPolicies](#overlapping-gateway-and-httproute-ratelimitpolicies).

=== UNIT ID: kuadrant-operator/doc/overviews/rate-limiting.md:targeting-a-gateway-networking-resource ===
DOC: kuadrant-operator/doc/overviews/rate-limiting.md
TITLE: Targeting a Gateway networking resource
BREADCRUMB: ['Kuadrant Rate Limiting', 'Using the RateLimitPolicy']
TOKENS: 1779
A RateLimitPolicy that targets a Gateway can declare a block of _defaults_ (`spec.defaults`) or a block of _overrides_ (`spec.overrides`). As a standard, gateway policies that do not specify neither defaults nor overrides, act as defaults.

When declaring _defaults_, a RateLimitPolicy which targets a Gateway will be enforced to all HTTP traffic hitting the gateway, unless a more specific RateLimitPolicy targeting a matching HTTPRoute or GRPCRoute exists. Any new route referencing the gateway as parent will be automatically covered by the default RateLimitPolicy, as well as changes in existing routes.

_Defaults_ provide cluster operators with the ability to protect the infrastructure against unplanned and malicious network traffic attempt, such as by setting safe default limits on hostnames and hostname wildcards.

Inversely, a gateway policy that specify _overrides_ declares a set of rules to be enforced on _all routes attached to the gateway_, thus atomically replacing any more specific policy occasionally attached to any of those routes.

Target a Gateway HTTPRoute by setting the `spec.targetRef` field of the RateLimitPolicy as follows:

```yaml
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: <RateLimitPolicy name>
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: <Gateway Name>
  defaults: # alternatively: `overrides`
    limits: { … }
```

![rate limit policy targeting a Gateway resource](../images/rlp-gateway-target.png)

#### Overlapping Gateway and HTTPRoute RateLimitPolicies

Two possible semantics are to be considered here – gateway policy _defaults_ vs gateway policy _overrides_.

Gateway RateLimitPolicies that declare _defaults_ (or alternatively neither defaults nor overrides) protect all traffic routed through the gateway except where a more specific HTTPRoute or GRPCRoute RateLimitPolicy exists, in which case the route-level RateLimitPolicy prevails.

Example with 4 RateLimitPolicies, 3 HTTPRoutes and 1 Gateway _default_ (plus 2 HTTPRoute and 2 Gateways without RateLimitPolicies attached):

- RateLimitPolicy A → HTTPRoute A (`a.toystore.com`) → Gateway G (`*.com`)
- RateLimitPolicy B → HTTPRoute B (`b.toystore.com`) → Gateway G (`*.com`)
- RateLimitPolicy W → HTTPRoute W (`*.toystore.com`) → Gateway G (`*.com`)
- RateLimitPolicy G (defaults) → Gateway G (`*.com`)

Expected behavior:

- Request to `a.toystore.com` → RateLimitPolicy A will be enforced
- Request to `b.toystore.com` → RateLimitPolicy B will be enforced
- Request to `other.toystore.com` → RateLimitPolicy W will be enforced
- Request to `other.com` (suppose a route exists) → RateLimitPolicy G will be enforced
- Request to `yet-another.net` (suppose a route and gateway exist) → No RateLimitPolicy will be enforced

Gateway RateLimitPolicies that declare _overrides_ protect all traffic routed through the gateway, regardless of existence of any more specific HTTPRoute RateLimitPolicy.

Example with 4 RateLimitPolicies, 3 HTTPRoutes and 1 Gateway _override_ (plus 2 HTTPRoute and 2 Gateways without RateLimitPolicies attached):

- RateLimitPolicy A → HTTPRoute A (`a.toystore.com`) → Gateway G (`*.com`)
- RateLimitPolicy B → HTTPRoute B (`b.toystore.com`) → Gateway G (`*.com`)
- RateLimitPolicy W → HTTPRoute W (`*.toystore.com`) → Gateway G (`*.com`)
- RateLimitPolicy G (overrides) → Gateway G (`*.com`)

Expected behavior:

- Request to `a.toystore.com` → RateLimitPolicy G will be enforced
- Request to `b.toystore.com` → RateLimitPolicy G will be enforced
- Request to `other.toystore.com` → RateLimitPolicy G will be enforced
- Request to `other.com` (suppose a route exists) → RateLimitPolicy G will be enforced
- Request to `yet-another.net` (suppose a route and gateway exist) → No RateLimitPolicy will be enforced

## Limit definition

A limit will be activated whenever a request comes in and the request matches:

- all of the `when` conditions specified in the limit.

A limit can define:

- counters that are qualified based on dynamic values fetched from the request, or
- global counters (implicitly, when no qualified counter is specified)

A limit is composed of one or more rate limits.

E.g.

```yaml
spec:
  limits:
    "toystore-all":
      rates:
        - limit: 5000
          window: 1s

    "toystore-api-per-username":
      rates:
        - limit: 100
          window: 1s
        - limit: 1000
          window: 1m
      counters:
        - expression: auth.identity.username
      when:
        - predicate: request.host == 'api.toystore.com'

    "toystore-admin-unverified-users":
      rates:
        - limit: 250
          window: 1s
      when:
        - predicate: request.host == 'admin.toystore.com'
        - predicate: !auth.identity.email_verified
```

| Request to           | Rate limits enforced                                         |
| -------------------- | ------------------------------------------------------------ |
| `api.toystore.com`   | 100rps/username or 1000rpm/username (whatever happens first) |
| `admin.toystore.com` | 250rps                                                       |
| `other.toystore.com` | 5000rps                                                      |

#### `when` conditions

`when` conditions can be used to scope a limit (i.e. to filter the traffic to which a limit definition applies) without any coupling to the underlying network topology, i.e. without making direct references to HTTPRouteRules.

Use `when` conditions to conditionally activate limits based on attributes that cannot be expressed in the HTTPRoutes' `spec.hostnames` and `spec.rules.matches` fields, or in general in RateLimitPolicies that target a Gateway.

The selectors within the `when` conditions of a RateLimitPolicy are a subset of Kuadrant's Well-known Attributes ([RFC 0002](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md)). Check out the reference for the full list of supported selectors.

## Examples

Check out the following user guides for examples of rate limiting services with Kuadrant:

- [Simple Rate Limiting for Applications](../user-guides/ratelimiting/simple-rl-for-app-developers.md)
- [Authenticated Rate Limiting for Application](../user-guides/ratelimiting/authenticated-rl-for-app-developers.md)
- [Gateway Rate Limiting for Cluster Operators](../user-guides/ratelimiting/gateway-rl-for-cluster-operators.md)
- [Authenticated Rate Limiting with JWTs and Kubernetes RBAC](../user-guides/ratelimiting/authenticated-rl-with-jwt-and-k8s-authnz.md)
- [Authenticated Rate Limiting for gRPC Services](../user-guides/ratelimiting/authenticated-rl-for-app-developers-grpc-services.md)

## Known limitations

- RateLimitPolicies can only target HTTPRoutes/GRPCRoutes/Gateways defined within the same namespace of the RateLimitPolicy.
- Mixed HTTPRoute and GRPCRoute sharing the same hostname on a Gateway is not supported. Gateway implementations enforce [hostname uniqueness](https://gateway-api.sigs.k8s.io/concepts/api-overview/#hostname-matching) across route types — use separate hostnames for HTTP and gRPC traffic.

=== UNIT ID: kuadrant-operator/doc/overviews/rate-limiting.md:implementation-details ===
DOC: kuadrant-operator/doc/overviews/rate-limiting.md
TITLE: Implementation details
BREADCRUMB: ['Kuadrant Rate Limiting']
TOKENS: 1787
Driven by limitations related to how Istio injects configuration in the filter chains of the ingress gateways, Kuadrant relies on Envoy's [Wasm Network](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/wasm_filter) filter in the data plane, to manage the integration with rate limiting service ("Limitador"), instead of the [Rate Limit](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter) filter.

**Motivation:** _Multiple rate limit domains_

The first limitation comes from having only one filter chain per listener. This often leads to one single global rate limiting filter configuration per gateway, and therefore to a shared rate limit [domain](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/ratelimit/v3/rate_limit.proto#envoy-v3-api-msg-extensions-filters-http-ratelimit-v3-ratelimit) across applications and policies. Even though, in a rate limit filter, the triggering of rate limit calls, via [actions to build so-called "descriptors"](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter#composing-actions), can be defined at the level of the virtual host and/or specific route rule, the overall rate limit configuration is only one, i.e., always the same rate limit domain for all calls to Limitador.

On the other hand, the possibility to configure and invoke the rate limit service for multiple domains depending on the context allows to isolate groups of policy rules, as well as to optimize performance in the rate limit service, which can rely on the domain for indexation.

**Motivation:** _Fine-grained matching rules_
A second limitation of configuring the rate limit filter via Istio, particularly from [Gateway API](https://gateway-api.sigs.k8s.io) resources, is that [rate limit descriptors](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter#composing-actions) at the level of a specific HTTP route rule require "named routes" – defined only in an Istio [VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/#HTTPRoute) resource and referred in an [EnvoyFilter](https://istio.io/latest/docs/reference/config/networking/envoy-filter/#EnvoyFilter-RouteConfigurationMatch-RouteMatch) one. Because Gateway API HTTPRoute rules lack a "name" property[^1], as well as the Istio VirtualService resources are only ephemeral data structures handled by Istio in-memory in its implementation of gateway configuration for Gateway API, where the names of individual route rules are auto-generated and not referable by users in a policy[^2][^3], rate limiting by attributes of the HTTP request (e.g., path, method, headers, etc) would be very limited while depending only on Envoy's [Rate Limit](https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/rate_limit_filter) filter.

[^1]: https://github.com/kubernetes-sigs/gateway-api/pull/996
[^2]: https://github.com/istio/istio/issues/36790
[^3]: https://github.com/istio/istio/issues/37346

Motivated by the desire to support multiple rate limit domains per ingress gateway, as well as fine-grained HTTP route matching rules for rate limiting, Kuadrant implements a [wasm-shim](https://github.com/Kuadrant/wasm-shim) that handles the rules to invoke the rate limiting service, complying with Envoy's [Rate Limit Service (RLS)](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto) protocol.

The wasm module integrates with the gateway in the data plane via [Wasm Network](https://www.envoyproxy.io/docs/envoy/latest/configuration/listeners/network_filters/wasm_filter) filter, and parses a configuration composed out of user-defined RateLimitPolicy resources by the Kuadrant control plane. Whereas the rate limiting service ("Limitador") remains an implementation of Envoy's RLS protocol, capable of being integrated directly via [Rate Limit](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/network/ratelimit/v3/rate_limit.proto#extension-envoy-filters-network-ratelimit) extension or by Kuadrant, via wasm module for the [Istio Gateway API implementation](https://gateway-api.sigs.k8s.io/implementations/#istio).

As a consequence of this design:

- Users can define fine-grained rate limit rules that match their Gateway and HTTPRoute definitions including for subsections of these.
- Rate limit definitions are insulated, not leaking across unrelated policies or applications.
- Conditions to activate limits are evaluated in the context of the gateway process, reducing the gRPC calls to the external rate limiting service only to the cases where rate limit counters are known in advance to have to be checked/incremented.
- The rate limiting service can rely on the indexation to look up for groups of limit definitions and counters.
- Components remain compliant with industry protocols and flexible for different integration options.

A Kuadrant wasm-shim configuration for one RateLimitPolicy custom resources targeting a HTTPRoute looks like the following and it is generated automatically by the Kuadrant control plane:

```yaml
apiVersion: extensions.istio.io/v1alpha1
kind: WasmPlugin
metadata:
  creationTimestamp: "2024-10-01T16:59:40Z"
  generation: 1
  name: kuadrant-kuadrant-ingressgateway
  namespace: gateway-system
  ownerReferences:
    - apiVersion: gateway.networking.k8s.io/v1
      blockOwnerDeletion: true
      controller: true
      kind: Gateway
      name: kuadrant-ingressgateway
      uid: 0298355b-fb30-4442-af2b-88d0c05bd2bd
  resourceVersion: "11253"
  uid: 36ef1fb7-9eca-46c7-af63-fe783f40148c
spec:
  phase: STATS
  pluginConfig:
    services:
      ratelimit-service:
        type: ratelimit
        endpoint: ratelimit-cluster
        failureMode: allow
    actionSets:
      - name: some_name_0
        routeRuleConditions:
          hostnames:
            - "*.toystore.website"
            - "*.toystore.io"
          predicates:
            - request.url_path.startsWith("/assets")
        actions:
          - service: ratelimit-service
            scope: gateway-system/app-rlp
            predicates:
              - request.host.endsWith('.toystore.website')
            data:
              - expression:
                  key: limit.toystore_assets_all_domains__b61ee8e6
                  value: "1"
      - name: some_name_1
        routeRuleConditions:
          hostnames:
            - "*.toystore.website"
            - "*.toystore.io"
          predicates:
            - request.url_path.startsWith("/v1")
        actions:
          - service: ratelimit-service
            scope: gateway-system/app-rlp
            predicates:
              - request.host.endsWith('.toystore.website')
              - auth.identity.username == ""
            data:
              - expression:
                  key: limit.toystore_v1_website_unauthenticated__377837ee
                  value: "1"
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: kuadrant-ingressgateway
  url: quay.io/kuadrant/wasm-shim:latest
```

=== UNIT ID: kuadrant-operator/doc/overviews/token-rate-limiting.md:token-tracking-protocol ===
DOC: kuadrant-operator/doc/overviews/token-rate-limiting.md
TITLE: Token Tracking Protocol
BREADCRUMB: ['Kuadrant Token Rate Limiting', 'How it works']
TOKENS: 1910
## Kuadrant Token Rate Limiting

A Kuadrant TokenRateLimitPolicy custom resource enables token-based rate limiting for AI/LLM workloads in a Gateway API network:

1. Targets Gateway API networking resources such as [HTTPRoutes](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.HTTPRoute) and [Gateways](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1.Gateway)
2. Automatically tracks actual token consumption from OpenAI-compatible API responses
3. Supports user segmentation and sophisticated limiting strategies
4. Integrates with AuthPolicy for user-based rate limiting using authentication claims
5. Enables cluster operators to set defaults that govern behaviour at the lower levels of the network, until a more specific policy is applied

Kuadrant's Token Rate Limit implementation extends the Envoy [Rate Limit Service (RLS)](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto) protocol with automatic token usage extraction. The workflow per request goes:

1. On incoming request, the gateway evaluates matching rules and predicates from TokenRateLimitPolicy resources
2. If the request matches, the gateway prepares rate limit descriptors and monitors the response
3. After receiving the response, the gateway extracts `usage.total_tokens` from the response body
4. The gateway sends a [RateLimitRequest](https://www.envoyproxy.io/docs/envoy/latest/api-v3/service/ratelimit/v3/rls.proto#service-ratelimit-v3-ratelimitrequest) to Limitador with the actual token count as `hits_addend`
5. Limitador tracks the cumulative token usage and responds with either `OK` or `OVER_LIMIT`

This approach ensures accurate usage-based rate limiting where limits are enforced based on actual AI/LLM token consumption rather than simple request counts.

**Important**: TokenRateLimitPolicy supports both non-streaming and streaming OpenAI-style API responses. For streaming, the request must include `"stream": true` and `"stream_options": { "include_usage": true }` for usage to be extracted from the final stream event. Only OpenAI-style completions responses are supported today — this includes `/v1/chat/completions` and `/v1/completions`, and any backend implementing the OpenAI-compatible API such as vLLM and kServe. Other provider formats (e.g. Anthropic, Google Gemini) are not yet parsed; see [#1864](https://github.com/Kuadrant/kuadrant-operator/issues/1864).

## The TokenRateLimitPolicy custom resource

#### Overview

The `TokenRateLimitPolicy` spec includes, basically, two parts:

- A reference to an existing Gateway API resource (`spec.targetRef`)
- Limit definitions (`spec.limits`)

Each limit definition includes:

- A set of rate limits (`spec.limits.<limit-name>.rates[]`)
- (Optional) A set of dynamic counter qualifiers (`spec.limits.<limit-name>.counters[]`)
- (Optional) A set of additional dynamic conditions to activate the limit (`spec.limits.<limit-name>.when[]`)

The limit definitions (`limits`) can be declared at the top-level level of the spec (with the semantics of _defaults_) or alternatively within explicit `defaults` or `overrides` blocks.

<table>
  <tbody>
    <tr>
      <td>Check out Kuadrant <a href="https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md">RFC 0002</a> to learn more about the Well-known Attributes that can be used to define counter qualifiers (<code>counters</code>) and conditions (<code>when</code>).</td>
    </tr>
  </tbody>
</table>

Check out the [API reference](../reference/tokenratelimitpolicy.md) for a full specification of the TokenRateLimitPolicy CRD.

## Automatic Token Tracking

TokenRateLimitPolicy automatically extracts token usage from AI/LLM responses without requiring any additional configuration:

- **Zero configuration**: Works out-of-the-box with OpenAI-compatible APIs
- **Response parsing**: Automatically extracts `usage.total_tokens` from response bodies
- **Provider scope**: Supports any backend returning that field, e.g. OpenAI `/v1/chat/completions` and `/v1/completions`, and OpenAI-compatible backends like vLLM, kServe, Ollama, Azure OpenAI, and Gemini's OpenAI-compat endpoint. Anthropic and Gemini's native response formats aren't supported yet — see [#1864](https://github.com/Kuadrant/kuadrant-operator/issues/1864)
- **Accurate accounting**: Tracks actual token consumption, not estimates
- **Graceful fallback**: If token parsing fails, falls back to request counting

## User Segmentation

Create different limits for different user tiers, organisations, or teams:

```yaml
limits:
  free-tier:
    rates:
    - limit: 20000
      window: 24h
    when:
    - predicate: 'auth.identity.groups.split(",").exists(g, g == "free")'
    counters:
    - expression: auth.identity.userid
```

## Multiple Time Windows

Protect against both burst usage and sustained overconsumption:

```yaml
limits:
  burst-protection:
    rates:
    - limit: 1000     # 1k tokens per minute (burst protection)
      window: 1m
    - limit: 50000    # 50k tokens per hour (sustained usage)
      window: 1h
    - limit: 500000   # 500k tokens per day (daily quota)
      window: 1d
```

## Policy Hierarchy and Precedence

TokenRateLimitPolicy supports three modes of operation that provide different levels of precedence:

## Implicit Defaults (using `limits`)

When a policy specifies `limits` directly at the spec level, these act as **implicit defaults**:
- Applied to the target resource (`Gateway` or `HTTPRoute`) 
- When targeting a Gateway: Can be overridden by more specific policies targeting individual routes
- Most common usage pattern for single-policy scenarios

## Explicit Defaults (using `defaults`)

When a policy uses the `defaults` field:
- Applied as default rules for routes that lack more specific policies
- Useful for Gateway-level policies that provide baseline limits  
- Can be overridden by HTTPRoute-level policies or Gateway overrides
- Supports merge strategies (`atomic` or `merge`)

## Overrides (using `overrides`)

When a policy uses the `overrides` field:
- Takes precedence over all other policies in the hierarchy
- Cannot be overridden by more specific policies
- Only allowed for Gateway-targeted policies
- Useful for enforcing organisation-wide limits that cannot be bypassed

## AI/LLM API Protection

Protect your AI/LLM APIs from token exhaustion while ensuring fair usage across different user tiers:

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: llm-protection
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ai-gateway
  limits:
    free-users:
      rates:
      - limit: 10000    # 10k tokens per day for free tier
        window: 24h
      when:
      - predicate: 'auth.identity.subscription == "free"'
      counters:
      - expression: auth.identity.userid
    
    pro-users:
      rates:
      - limit: 100000   # 100k tokens per day for pro tier
        window: 24h
      when:
      - predicate: 'auth.identity.subscription == "pro"'
      counters:
      - expression: auth.identity.userid
```

## Organisation-Wide Quotas

Enforce organisation-level limits that cannot be bypassed by individual teams:

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: org-quotas
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: api-gateway
  overrides:
    limits:
      org-total:
        rates:
        - limit: 10000000  # 10M tokens per month org-wide
          window: 720h
        counters:
        - expression: auth.identity.org_id
```

=== UNIT ID: kuadrant-operator/doc/overviews/token-rate-limiting.md:integration-with-authpolicy ===
DOC: kuadrant-operator/doc/overviews/token-rate-limiting.md
TITLE: Integration with AuthPolicy
BREADCRUMB: ['Kuadrant Token Rate Limiting']
TOKENS: 442
TokenRateLimitPolicy works seamlessly with Kuadrant's AuthPolicy to enable user-based rate limiting. When using API key authentication with groups, ensure the AuthPolicy exposes the necessary attributes:

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: api-auth
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: api-gateway
  rules:
    authentication:
      api-key-users:
        apiKey:
          selector:
            matchLabels:
              app: my-llm
        credentials:
          authorizationHeader:
            prefix: APIKEY
    response:
      success:
        filters:
          identity:
            json:
              properties:
                groups:
                  selector: auth.identity.metadata.annotations.kuadrant\.io/groups
                userid:
                  selector: auth.identity.metadata.annotations.secret\.kuadrant\.io/user-id
    authorization:
      allow-groups:
        opa:
          rego: |
            groups := split(object.get(input.auth.identity.metadata.annotations, "kuadrant.io/groups", ""), ",")
            allow { groups[_] == "free" }
            allow { groups[_] == "gold" }
```

This configuration makes `auth.identity.groups` and `auth.identity.userid` available to TokenRateLimitPolicy for use in predicates and counters.

## See Also

- [TokenRateLimitPolicy API Reference](../reference/tokenratelimitpolicy.md)
- [Token Rate Limiting Tutorial](../user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md)
- [RateLimitPolicy Overview](rate-limiting.md) - For non-token-based rate limiting
- [AuthPolicy Overview](auth.md) - For authentication configuration
- [Gateway API Documentation](https://gateway-api.sigs.k8s.io/)

=== UNIT ID: kuadrant-operator/doc/reference/authpolicy.md:authpolicyspec ===
DOC: kuadrant-operator/doc/reference/authpolicy.md
TITLE: AuthPolicySpec
BREADCRUMB: ['The AuthPolicy Custom Resource Definition (CRD)']
TOKENS: 1622
## AuthPolicy

| **Field** | **Type**                              | **Required** | **Description**                                 |
|-----------|---------------------------------------|:------------:|-------------------------------------------------|
| `spec`    | [AuthPolicySpec](#authpolicyspec)     | Yes          | The specification for AuthPolicy custom resource |
| `status`  | [AuthPolicyStatus](#authpolicystatus) | No           | The status for the custom resource              |

| **Field**        | **Type**                                                                                                                                    | **Required** | **Description**                                                                                                                                                                                                                                                                                 |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `targetRef`      | [Gateway API LocalPolicyTargetReferenceWithSectionName](https://gateway-api.sigs.k8s.io/reference/spec/#localpolicytargetreferencewithsectionname) | Yes          | Reference to a Kubernetes resource that the policy attaches to                                                                                                                                                                                                                                  |
| `rules`          | [AuthScheme](#authscheme)                                                                                                                   | No           | Implicit default authentication/authorization rules                                                                                                                                                                                                                                             |
| `patterns`       | Map<String: [NamedPattern](#namedpattern)>                                                                                                  | No           | Implicit default named patterns of lists of `selector`, `operator` and `value` tuples, to be reused in `when` conditions and pattern-matching authorization rules.                                                                                                                              |
| `when`           | [][PatternExpressionOrRef](https://docs.kuadrant.io/latest/authorino/docs/features/#common-feature-conditions-when)                                | No           | List of implicit default additional dynamic conditions (expressions) to activate the policy. Use it for filtering attributes that cannot be expressed in the targeted route's `spec.hostnames` and `spec.rules.matches` fields, or when targeting a Gateway.                                |
| `defaults`       | [AuthPolicyCommonSpec](#authPolicyCommonSpec)                                                                                               | No           | Explicit default definitions. This field is mutually exclusive with any of the implicit default definitions: `spec.rules`, `spec.patterns`, `spec.when`                                                                                                                  |
| `overrides`      | [AuthPolicyCommonSpec](#authPolicyCommonSpec)                                                                                               | No           | Atomic overrides definitions. This field is mutually exclusive with any of the implicit or explicit default definitions: `spec.rules`, `spec.patterns`, `spec.when`, `spec.default`                                                                                      |

## AuthPolicyCommonSpec

| **Field**        | **Type**                                                                                                                                    | **Required** | **Description**                                                                                                                                                                                                                                                                |
|------------------|---------------------------------------------------------------------------------------------------------------------------------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `rules`          | [AuthScheme](#authscheme)                                                                                                                   | No           | Authentication/authorization rules                                                                                                                                                                                                                                             |
| `patterns`       | Map<String: [NamedPattern](#namedpattern)>                                                                                                  | No           | Named patterns of lists of `selector`, `operator` and `value` tuples, to be reused in `when` conditions and pattern-matching authorization rules.                                                                                                                              |
| `when`           | [][PatternExpressionOrRef](https://docs.kuadrant.io/latest/authorino/docs/features/#common-feature-conditions-when)                                | No           | List of additional dynamic conditions (expressions) to activate the policy. Use it for filtering attributes that cannot be expressed in the targeted route's `spec.hostnames` and `spec.rules.matches` fields, or when targeting a Gateway.                                |

=== UNIT ID: kuadrant-operator/doc/reference/authpolicy.md:authscheme ===
DOC: kuadrant-operator/doc/reference/authpolicy.md
TITLE: AuthScheme
BREADCRUMB: ['The AuthPolicy Custom Resource Definition (CRD)', 'AuthPolicyCommonSpec']
TOKENS: 5315
| **Field**        | **Type**                                               | **Required** | **Description**                                                                                                                                                             |
|------------------|--------------------------------------------------------|:------------:|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `authentication` | Map<String: [AuthenticationRule](#authenticationrule)> | No           | Authentication rules. At least one config MUST evaluate to a valid identity object for the auth request to be successful. If omitted or empty, anonymous access is assumed. |
| `metadata`       | Map<String: [MetadataRule](#metadatarule)>             | No           | Rules for fetching auth metadata from external sources.                                                                                                                     |
| `authorization`  | Map<String: [AuthorizationRule](#authorizationrule)>   | No           | Authorization rules. All policies MUST allow access for the auth request be successful.                                                                                     |
| `response`       | [ResponseSpec](#responsespec)                          | No           | Customizations to the response to the authorization request. Use it to set custom values for unauthenticated, unauthorized, and/or success access request.                  |
| `callbacks`      | Map<String: [CallbackRule](#callbackrule)>             | No           | Rules for post-authorization callback requests to external services. Triggered regardless of the result of the authorization request.                                       |

#### AuthRuleCommon

| **Field**               | **Type**                                                                                                     | **Required** | **Description**                                                                                                                                                                                                                                                                             |
|-------------------------|--------------------------------------------------------------------------------------------------------------|:------------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `when`                  | [][PatternExpressionOrRef](https://docs.kuadrant.io/latest/authorino/docs/features/#common-feature-conditions-when) | No           | List of additional dynamic conditions (expressions) to activate the auth rule. Use it for filtering attributes that cannot be expressed in the targeted route's `spec.hostnames` and `spec.rules.matches` fields, or when targeting a Gateway.                                          |
| `cache`                 | [Caching spec](https://docs.kuadrant.io/latest/authorino/docs/features/#common-feature-caching-cache)               | No           | Caching options for the resolved object returned when applying this auth rule. (Default: disabled)                                                                                                                                                                                          |
| `priority`              | Integer                                                                                                      | No           | Priority group of the auth rule. All rules in the same priority group are evaluated concurrently; consecutive priority groups are evaluated sequentially. (Default: `0`)                                                                                                                    |
| `metrics`               | Boolean                                                                                                      | No           | Whether the auth rule emits individual observability metrics. (Default: `false`)                                                                                                                                                                                                            |

#### AuthenticationRule

| **Field**               | **Type**                                                                                                                                                 | **Required** | **Description**                                                                                                                                                                                                                                                                                         |
|-------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|:------------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `apiKey`                | [API Key authentication spec](https://docs.kuadrant.io/latest/authorino/docs/features/#api-key-authenticationapikey)                                            | No           | Authentication based on API keys stored in Kubernetes secrets. Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`.                                                                                                                               |
| `kubernetesTokenReview` | [KubernetesTokenReview spec](https://docs.kuadrant.io/latest/authorino/docs/features/#kubernetes-tokenreview-authenticationkubernetestokenreview)               | No           | Authentication by Kubernetes token review. Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`.                                                                                                                                                   |
| `jwt`                   | [JWT verification spec](https://docs.kuadrant.io/latest/authorino/docs/features/#jwt-verification-authenticationjwt)                                            | No           | Authentication based on JSON Web Tokens (JWT). Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`.                                                                                                                                               |
| `oauth2Introspection`   | [OAuth2 Token Introscpection spec](https://docs.kuadrant.io/latest/authorino/docs/features/#oauth-20-introspection-authenticationoauth2introspection)           | No           | Authentication by OAuth2 token introspection. Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`.                                                                                                                                                |
| `x509`                  | [X.509 authentication spec](https://docs.kuadrant.io/latest/authorino/docs/features/#x509-client-certificate-authentication-authenticationx509)                 | No           | Authentication based on client X.509 certificates. The certificates presented by the clients must be signed by a trusted CA whose certificates are stored in Kubernetes secrets. Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`.             |
| `plain`                 | [Plain identity object spec](https://docs.kuadrant.io/latest/authorino/docs/features/#plain-authenticationplain)                                                | No           | Identity object extracted from the context. Use this method when authentication is performed beforehand by a proxy and the resulting object passed to Authorino as JSON in the auth request. Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`. |
| `anonymous`             | [Anonymous access](https://docs.kuadrant.io/latest/authorino/docs/features/#anonymous-access-authenticationanonymous)                                           | No           | Anonymous access. Use one of: `apiKey`, `jwt`, `oauth2Introspection`, `kubernetesTokenReview`, `x509`, `plain`, `anonymous`.                                                                                                                                                                            |
| `credentials`           | [Auth credentials spec](https://docs.kuadrant.io/latest/authorino/docs/features/#extra-auth-credentials-authenticationcredentials)                              | No           | Customizations to where credentials are required to be passed in the request for authentication based on this auth rule. Defaults to HTTP Authorization header with prefix "Bearer".                                                                                                                    |
| `overrides`             | [Identity extension spec](https://docs.kuadrant.io/latest/authorino/docs/features/#extra-identity-extension-authenticationdefaults-and-authenticationoverrides) | No           | JSON overrides to set to the resolved identity object. Do not use it with identity objects of other JSON types (array, string, etc).                                                                                                                                                                    |
| `defaults`              | [Identity extension spec](https://docs.kuadrant.io/latest/authorino/docs/features/#extra-identity-extension-authenticationdefaults-and-authenticationoverrides) | No           | JSON defaults to set to the resolved identity object. Do not use it with identity objects of other JSON types (array, string, etc).                                                                                                                                                                     |
| _(inline)_              | [AuthRuleCommon](#authrulecommon)                                                                                                                        | No           |                                                                                                                                                                                                                                                                                                         |

#### MetadataRule

| **Field**   | **Type**                                                                                                                          | **Required** | **Description**                                                                                                                         |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------|:------------:|-----------------------------------------------------------------------------------------------------------------------------------------|
| `http`      | [HTTP GET/GET-by-POST external metadata spec](https://docs.kuadrant.io/latest/authorino/docs/features/#http-getget-by-post-metadatahttp) | No           | External source of auth metadata via HTTP request. Use one of: `http`, `userInfo`, `uma`.                                               |
| `userInfo`  | [OIDC UserInfo spec](https://docs.kuadrant.io/latest/authorino/docs/features/#oidc-userinfo-metadatauserinfo)                            | No           | OpendID Connect UserInfo linked to an OIDC authentication rule declared in this same AuthPolicy. Use one of: `http`, `userInfo`, `uma`. |
| `uma`       | [UMA metadata spec](https://docs.kuadrant.io/latest/authorino/docs/features/#user-managed-access-uma-resource-registry-metadatauma)      | No           | User-Managed Access (UMA) source of resource data.  Use one of: `http`, `userInfo`, `uma`.                                              |
| _(inline)_  | [AuthRuleCommon](#authrulecommon)                                                                                                 | No           |                                                                                                                                         |

#### AuthorizationRule

| **Field**                       | **Type**                                                                                                                                                           | **Required** | **Description**                                                                                                                                        |
|---------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------:|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| `patternMatching`               | [Pattern-matching authorization spec](https://docs.kuadrant.io/latest/authorino/docs/features/#pattern-matching-authorization-authorizationpatternmatching)               | No           | Pattern-matching authorization rules. Use one of: `patternMatching`, `opa`, `kubernetesSubjectAccessReview`, `spicedb`.                                |
| `opa`                           | [OPA authorization spec](https://docs.kuadrant.io/latest/authorino/docs/features/#open-policy-agent-opa-rego-policies-authorizationopa)                                   | No           | Open Policy Agent (OPA) Rego policy. Use one of: `patternMatching`, `opa`, `kubernetesSubjectAccessReview`, `spicedb`.                                 |
| `kubernetesSubjectAccessReview` | [Kubernetes SubjectAccessReview spec](https://docs.kuadrant.io/latest/authorino/docs/features/#kubernetes-subjectaccessreview-authorizationkubernetessubjectaccessreview) | No           | Authorization by Kubernetes SubjectAccessReview. Use one of: `patternMatching`, `opa`, `kubernetesSubjectAccessReview`, `spicedb`.                     |
| `spicedb`                       | [SpiceDB authorization spec](https://docs.kuadrant.io/latest/authorino/docs/features/#spicedb-authorizationspicedb)                                                       | No           | Authorization decision delegated to external Authzed/SpiceDB server. Use one of: `patternMatching`, `opa`, `kubernetesSubjectAccessReview`, `spicedb`. |
| _(inline)_                      | [AuthRuleCommon](#authrulecommon)                                                                                                                                  | No           |                                                                                                                                                        |

#### ResponseSpec

| **Field**         | **Type**                                                                                                                                                           | **Required** | **Description**                                                                                                                    |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------:|------------------------------------------------------------------------------------------------------------------------------------|
| `unauthenticated` | [Custom denial status spec](https://docs.kuadrant.io/latest/authorino/docs/features/#custom-denial-status-responseunauthenticated-and-responseunauthorized)               | No           | Customizations on the denial status and other HTTP attributes when the request is unauthenticated. (Default: `401 Unauthorized`)   |
| `unauthorized`    | [Custom denial status spec](https://docs.kuadrant.io/latest/authorino/docs/features/#custom-denial-status-responseunauthenticated-and-responseunauthorized)               | No           | Customizations on the denial status and other HTTP attributes when the request is unauthorized. (Default: `403 Forbidden`)         |
| `success`         | [SuccessResponseSpec](#successresponsespec)                                                                                                                        | No           | Response items to be included in the auth response when the request is authenticated and authorized.                               |

##### SuccessResponseSpec

| **Field** | **Type**                                                 | **Required** | **Description**                                                                                      |
|-----------|----------------------------------------------------------|:------------:|------------------------------------------------------------------------------------------------------|
| `headers` | Map&lt;string:[SuccessResponseItem](#successresponseitem)&gt; |      No      | Custom success response items wrapped as HTTP headers to be injected in the request.                 |
| `filters` | Map&lt;string:[SuccessResponseItem](#successresponseitem)&gt; |      No      | Custom success response items made available to other filters managed by Kuadrant (i.e. Rate Limit). |

###### SuccessResponseItem

| **Field**   | **Type**                                                                                                                                                             | **Required** | **Description**                                                                                                                                                               |
|-------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `plain`     | [Plain text response item](https://docs.kuadrant.io/latest/authorino/docs/features/#plain-text-responsesuccessheadersdynamicmetadataplain)                                  | No           | Plain text content. Use one of: `plain`, `json`, `wristband`.                                                                                                                 |
| `json`      | [JSON injection response item](https://docs.kuadrant.io/latest/authorino/docs/features/#json-injection-responsesuccessheadersdynamicmetadatajson)                           | No           | Specification of a JSON object. Use one of: `plain`, `json`, `wristband`.                                                                                                     |
| `wristband` | [Festival Wristband token response item](https://docs.kuadrant.io/latest/authorino/docs/features/#festival-wristband-tokens-responsesuccessheadersdynamicmetadatawristband) | No           | Specification of a JSON object. Use one of: `plain`, `json`, `wristband`.                                                                                                     |
| `key`       | String                                                                                                                                                               | No           | The key used to add the custom response item (name of the HTTP header or root property of the Dynamic Metadata object). Defaults to the name of the response item if omitted. |

#### CallbackRule

| **Field**        | **Type**                                                                                                       | **Required** | **Description**                                                 |
|------------------|----------------------------------------------------------------------------------------------------------------|:------------:|-----------------------------------------------------------------|
| `http`           | [HTTP endpoints callback spec](https://docs.kuadrant.io/latest/authorino/docs/features/#http-endpoints-callbackshttp) | No           | HTTP endpoint settings to build the callback request (webhook). |
| _(inline)_       | [AuthRuleCommon](#authrulecommon)                                                                              | No           |                                                                 |

=== UNIT ID: kuadrant-operator/doc/reference/authpolicy.md:namedpattern ===
DOC: kuadrant-operator/doc/reference/authpolicy.md
TITLE: NamedPattern
BREADCRUMB: ['The AuthPolicy Custom Resource Definition (CRD)', 'AuthPolicyCommonSpec']
TOKENS: 1954
| **Field**  | **Type** | **Required** | **Description**                                                                                                                                                                                                    |
|------------|----------|:------------:|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `selector` | String   | Yes          | A valid [Well-known attribute](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md) whose resolved value in the data plane will be compared to `value`, using the `operator`.    |
| `operator` | String   | Yes          | The binary operator to be applied to the resolved value specified by the selector. One of: `eq` (equal to), `neq` (not equal to), `incl` (includes; for arrays), `excl` (excludes; for arrays), `matches` (regex). |
| `value`    | String   | Yes          | The static value to be compared to the one resolved from the selector.                                                                                                                                             |

## AuthPolicyStatus

| **Field**            | **Type**                          | **Description**                                                                                                                     |
|----------------------|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `observedGeneration` | String                            | Number of the last observed generation of the resource. Use it to check if the status info is up to date with latest resource spec. |
| `conditions`         | [][ConditionSpec](#conditionspec) | List of conditions that define that status of the resource.                                                                         |

## ConditionSpec

* The *lastTransitionTime* field provides a timestamp for when the entity last transitioned from one status to another.
* The *message* field is a human-readable message indicating details about the transition.
* The *reason* field is a unique, one-word, CamelCase reason for the condition’s last transition.
* The *status* field is a string, with possible values **True**, **False**, and **Unknown**.
* The *type* field is a string with the following possible values:
  * Available: the resource has successfully configured;

| **Field**            | **Type**  | **Description**              |
|----------------------|-----------|------------------------------|
| `type`               | String    | Condition Type               |
| `status`             | String    | Status: True, False, Unknown |
| `reason`             | String    | Condition state reason       |
| `message`            | String    | Condition state description  |
| `lastTransitionTime` | Timestamp | Last transition timestamp    |

#### High-level example

```yaml
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: my-auth-policy
spec:
  # Reference to an existing networking resource to attach the policy to. REQUIRED.
  # It can be a Gateway API HTTPRoute, GRPCRoute, or Gateway resource.
  # It can only refer to objects in the same namespace as the AuthPolicy.
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute # or GRPCRoute, Gateway
    name: myroute

  # Additional dynamic conditions to trigger the AuthPolicy.
  # Use it for filtering attributes not supported by the targeted route's rules or with AuthPolicies that target a Gateway.
  # Check out https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md to learn more
  # about the Well-known Attributes that can be used in this field.
  # Equivalent to if otherwise declared within `defaults`.
  when: […]

  # Sets of common patterns of selector-operator-value triples, to be referred by name in `when` conditions
  # and pattern-matching rules. Often employed to avoid repetition in the policy.
  # Equivalent to if otherwise declared within `defaults`.
  patterns: { … }

  # The auth rules to apply to the network traffic routed through the targeted resource.
  # Equivalent to if otherwise declared within `defaults`.
  rules:
    # Authentication rules to enforce.
    # At least one config must evaluate to a valid identity object for the auth request to be successful.
    # If omitted or empty, anonymous access is assumed.
    authentication:
      "my-authn-rule":
        # The authentication method of this rule.
        # One-of: apiKey, jwt, oauth2Introspection, kubernetesTokenReview, x509, plain, anonymous.
        apiKey: { … }

        # Where credentials are required to be passed in the request for authentication based on this rule.
        # One-of: authorizationHeader, customHeader, queryString, cookie.
        credentials:
          authorizationHeader:
            prefix: APIKEY

        # Rule-level additional conditions.
        when: […]

        # Configs for caching the resolved object returned out of evaluating this auth rule.
        cache: { … }

    # Rules for fetching auth metadata from external sources.
    metadata:
      "my-external-source":
        # The method for fetching metadata from the external source.
        # One-of: http: userInfo, uma.
        http: { … }

    # Authorization rules to enforce.
    # All policies must allow access for the auth request be successful.
    authorization:
      "my-authz-rule":
        # The authorization method of this rule.
        # One-of: patternMatching, opa, kubernetesSubjectAccessReview, spicedb.
        opa: { … }

    # Customizations to the authorization response.
    response:
      # Custom denial status and other HTTP attributes for unauthenticated requests.
      unauthenticated: { … }

      # Custom denial status and other HTTP attributes for unauhtorized requests.
      unauthorized: { … }

      # Custom response items when access is granted.
      success:
        # Custom response items wrapped as HTTP headers to be injected in the request
        headers:
          "my-custom-header":
            # One-of: plain, json, wristband.
            plain: { … }

        # Custom response items wrapped as envoy dynamic metadata.
        dynamicMetadata:
          # One-of: plain, json, wristband.
          "my-custom-dyn-metadata":
            json: { … }

    # Rules for post-authorization callback requests to external services.
    # Triggered regardless of the result of the authorization request.
    callbacks:
      "my-webhook":
        http: { … }

    # Explicit defaults. Used in policies that target a Gateway object to express default rules to be enforced on
    # routes that lack a more specific policy attached to.
    # Mutually exclusive with `overrides` and with declaring the `rules`, `when` and `patterns` at the top-level of
    # the spec.
    defaults:
      rules:
        authentication: { … }
        metadata: { … }
        authorization: { … }
        response: { … }
        callbacks: { … }
      when: […]
      patterns: { … }

    # Overrides. Used in policies that target a Gateway object to be enforced on all routes linked to the gateway,
    # thus also overriding any more specific policy occasionally attached to any of those routes.
    # Mutually exclusive with `defaults` and with declaring `rules`, `when` and `patterns` at the top-level of
    # the spec.
    overrides:
      rules:
        authentication: { … }
        metadata: { … }
        authorization: { … }
        response: { … }
        callbacks: { … }
      when: […]
      patterns: { … }
```

=== UNIT ID: kuadrant-operator/doc/reference/kuadrant.md:kuadrantspec ===
DOC: kuadrant-operator/doc/reference/kuadrant.md
TITLE: KuadrantSpec
BREADCRUMB: ['The Kuadrant Custom Resource Definition (CRD)', 'kuadrant']
TOKENS: 1784
## kuadrant

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `spec`    | [KuadrantSpec](#kuadrantspec)     |      No      | Blank specification                  |
| `status`  | [KuadrantStatus](#kuadrantstatus) |      No      | The status for the custom resources. |

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `observability`    | [Observability](#observability)     | No | Kuadrant observability configuration. |
| `mtls`  | [mTLS](#mtls) |      No      | Two way authentication between kuadrant components. |
| `components`  | [Components](#components) |      No      | Optional Kuadrant components configuration. |

#### mTLS

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `enable`    | Boolean     |  No | Enable mutual authentication communication between the gateway and the kuadrant data plane components. Default: `false`|
| `limitador` | Boolean     |  No | Enable mutual authentication communication between the gateway and Limitador. Default: `not set`|
| `authorino` | Boolean     |  No | Enable mutual authentication communication between the gateway and Authorino. Default: `not set`|

The truth table for limitador component is as follows:

| Spec |  Limtador mTLS enabled |
| --- | ---  |
| `{Enable: false, limitador: null}` | false |
| `{Enable: true, limitador: null}` | true |
| `{Enable: false, limitador: false}` | false |
| `{Enable: false, limitador: true}` | false |
| `{Enable: true, limitador: false}` | false |
| `{Enable: true, limitador: true}` | true |

The truth table for authorino component is as follows:

| Spec |  Authorino mTLS enabled |
| --- | ---  |
| `{Enable: false, authorino: null}` | false |
| `{Enable: true, authorino: null}` | true |
| `{Enable: false, authorino: false}` | false |
| `{Enable: false, authorino: true}` | false |
| `{Enable: true, authorino: false}` | false |
| `{Enable: true, authorino: true}` | true |

#### Observability

Configures telemetry and monitoring settings for Kuadrant components. When enabled, it configures logging, tracing, and other observability features for both the control plane and data plane components.

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `enable`    | Boolean     |  No | Enable observability on kuadrant. Default: `false` |
| `dataPlane` | [DataPlane](#dataplane) | No | Configures observability settings for the data plane components. |
| `tracing` | [Tracing](#tracing) | No | Configures distributed tracing for request flows through the system. |

##### DataPlane

Configures observability for data plane components (WASM filters). Controls OpenTelemetry trace filtering and request correlation features.

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `defaultLevels` | [][LogLevel](#loglevel) | No | Specifies the OpenTelemetry trace filtering levels for WASM modules. Controls which trace spans are exported to your observability backend (Jaeger, Tempo, etc.). The highest priority level set determines the filter level. **Important:** This controls trace span filtering, not gateway pod log verbosity. To control logs visible via `kubectl logs`, configure Envoy's log level separately. |
| `httpHeaderIdentifier` | String | No | Specifies the HTTP header name used to identify and correlate requests in traces (e.g., "x-request-id", "x-correlation-id"). If set, this header value will be included in trace spans for request correlation across components. |

###### LogLevel

Defines a trace filtering level. The highest priority level set (DEBUG > INFO > WARN > ERROR) determines the OTEL trace filter. Only one field should be set per LogLevel entry.

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `debug` | String | No | Debug level for trace filtering - highest verbosity, most spans exported. **Current implementation (MVP):** Set to `"true"` to enable. **Future:** Will support CEL expressions for dynamic request-time evaluation. |
| `info` | String | No | Info level for trace filtering. **Current implementation (MVP):** Set to `"true"` to enable. **Future:** Will support CEL expressions. |
| `warn` | String | No | Warn level for trace filtering. **Current implementation (MVP):** Set to `"true"` to enable. **Future:** Will support CEL expressions. |
| `error` | String | No | Error level for trace filtering - lowest verbosity, minimal spans exported. **Current implementation (MVP):** Set to `"true"` to enable. **Future:** Will support CEL expressions. |

**Example:**
```yaml
dataPlane:
  defaultLevels:
    - debug: "true"  # Enable DEBUG level trace filtering
  httpHeaderIdentifier: x-request-id
```

##### Tracing

Configures distributed tracing integration for request flows. It enables tracing spans to be exported to external tracing systems (e.g., Jaeger, Zipkin, Tempo).

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `defaultEndpoint` | String | No | The default URL of the tracing collector backend where spans should be sent. This endpoint is used by Auth (Authorino), RateLimiting (Limitador) and WASM services for exporting trace data. If tracing endpoints have been configured directly in Authorino or Limitador CRs, those take precedence over this default value. Note: Per-gateway overrides are not currently supported. |
| `insecure` | Boolean | No | Controls whether to skip TLS certificate verification. Default: `false` |

#### Components

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `developerPortal`    | [DeveloperPortal](#developerportal)     |  No | Developer portal integration configuration. |

##### DeveloperPortal

| **Field** | **Type**                          | **Required** | **Description**                      |
|-----------|-----------------------------------|:------------:|--------------------------------------|
| `enabled`    | Boolean     |  No | Enable the developer portal integration including APIProduct and APIKeyRequest CRDs. Default: `false` |

=== UNIT ID: kuadrant-operator/doc/reference/kuadrant.md:kuadrantstatus ===
DOC: kuadrant-operator/doc/reference/kuadrant.md
TITLE: KuadrantStatus
BREADCRUMB: ['The Kuadrant Custom Resource Definition (CRD)', 'kuadrant']
TOKENS: 282
| **Field**            | **Type**                                                                                     | **Description**                                                                                                                     |
|----------------------|----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `observedGeneration` | String                                                                                       | Number of the last observed generation of the resource. Use it to check if the status info is up to date with latest resource spec. |
| `conditions`         | [][ConditionSpec](https://pkg.go.dev/k8s.io/apimachinery@v0.28.4/pkg/apis/meta/v1#Condition) | List of conditions that define that status of the resource.                                                                         |
| `mtlsLimitador` | Boolean | Limitador mTLS enabled. |
| `mtlsAuthorino` | Boolean | Authorino mTLS enabled. |

=== UNIT ID: kuadrant-operator/doc/reference/ratelimitpolicy.md:ratelimitpolicyspec ===
DOC: kuadrant-operator/doc/reference/ratelimitpolicy.md
TITLE: RateLimitPolicySpec
BREADCRUMB: ['The RateLimitPolicy Custom Resource Definition (CRD)']
TOKENS: 1672
## RateLimitPolicy

| **Field** | **Type**                                        | **Required** | **Description**                                       |
|-----------|-------------------------------------------------|:------------:|-------------------------------------------------------|
| `spec`    | [RateLimitPolicySpec](#ratelimitpolicyspec)     |     Yes      | The specification for RateLimitPolicy custom resource |
| `status`  | [RateLimitPolicyStatus](#ratelimitpolicystatus) |      No      | The status for the custom resource                    |

| **Field**   | **Type**                                                                                                                                    | **Required** | **Description**                                                                                                                                                                             |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `targetRef` | [LocalPolicyTargetReferenceWithSectionName](#localpolicytargetreferencewithsectionname) | Yes          | Reference to a Kubernetes resource that the policy attaches to. For more [info](https://gateway-api.sigs.k8s.io/reference/spec/#localpolicytargetreferencewithsectionname)                                                                                                                              |
| `defaults`  | [RateLimitPolicyCommonSpec](#rateLimitPolicyCommonSpec)                                                                                     | No           | Default limit definitions. This field is mutually exclusive with the `limits` field                                                                                                         |
| `overrides` | [RateLimitPolicyCommonSpec](#rateLimitPolicyCommonSpec)                                                                                     | No           | Overrides limit definitions. This field is mutually exclusive with the `limits` field and `defaults` field. This field is only allowed for policies targeting `Gateway` in `targetRef.kind` |
| `limits`    | Map<String: [Limit](#limit)>                                                                                                                | No           | Limit definitions. This field is mutually exclusive with the [`defaults`](#rateLimitPolicyCommonSpec) field                                                                                 |

## LocalPolicyTargetReferenceWithSectionName

| **Field**       | **Type**                                | **Required** | **Description**                                            |
|------------------|-----------------------------------------|--------------|------------------------------------------------------------|
| `LocalPolicyTargetReference`         | [LocalPolicyTargetReference](#localpolicytargetreference)          | Yes          | Reference to a local policy target.               |
| `sectionName`    | [SectionName](#sectionname)                         | No           | Section name for further specificity (if needed). |

## LocalPolicyTargetReference

| **Field** | **Type**     | **Required** | **Description**                |
|-----------|--------------|--------------|--------------------------------|
| `group`   | `Group`      | Yes          | Group of the target resource. |
| `kind`    | `Kind`       | Yes          | Kind of the target resource.  |
| `name`    | `ObjectName` | Yes          | Name of the target resource.  |

## SectionName

| Field       | Type                     | Required | Description                                                                                                                                                                                                                         |
|-------------|--------------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SectionName | v1.SectionName (String)  | Yes      | SectionName is the name of a section in a Kubernetes resource. <br>In the following resources, SectionName is interpreted as the following: <br>- Gateway: Listener name<br>- HTTPRoute: HTTPRouteRule name<br>- GRPCRoute: GRPCRouteRule name<br>- Service: Port name |

## RateLimitPolicyCommonSpec

| **Field** | **Type**                     | **Required** | **Description**                                                                                                              |
|-----------|------------------------------|--------------|------------------------------------------------------------------------------------------------------------------------------|
| `when`    | [][Predicate](#predicate)    | No           | List of dynamic predicates to activate the policy. All expression must evaluate to true for the policy to be applied         |
| `limits`  | Map<String: [Limit](#limit)> | No           | Explicit Limit definitions. This field is mutually exclusive with [RateLimitPolicySpec](#ratelimitpolicyspec) `limits` field |

## Predicate

| **Field** | **Type**                     | **Required** | **Description**                                                                                                              |
|----------------|-------------------------|--------------|------------------------------------------------------------------------------------------------------------------------------|
| `predicate`    | String                  | Yes          | Defines one CEL expression that must be evaluated to bool                                                                    |

## Counter

| **Field** | **Type**                     | **Required** | **Description**                                                                                                               |
|-----------------|-------------------------|--------------|------------------------------------------------------------------------------------------------------------------------------|
| `expression`    | String                  | Yes          | Defines one CEL expression that will be used as rate limiting counter                                                        |

=== UNIT ID: kuadrant-operator/doc/reference/ratelimitpolicy.md:limit ===
DOC: kuadrant-operator/doc/reference/ratelimitpolicy.md
TITLE: Limit
BREADCRUMB: ['The RateLimitPolicy Custom Resource Definition (CRD)', 'RateLimitPolicySpec']
TOKENS: 1683
| **Field**        | **Type**                                            | **Required** | **Description**                                                                                                                                                                                                                                                                                                  |
|------------------|-----------------------------------------------------|:------------:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `rates`          | [][RateLimit](#ratelimit)                           |      No      | List of rate limits associated with the limit definition                                                                                                                                                                                                                                                         |
| `counters`       | [][Counter](#counter)                               |      No      | List of rate limit counter qualifiers. Items must be a valid [Well-known attribute](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md). Each distinct value resolved in the data plane starts a separate counter for each rate limit.                                        |
| `when`           | [][Predicate](#predicate)                           |      No      | List of dynamic predicates to activate the limit. All expression must evaluate to true for the limit to be applied                                                                        |

#### RateLimit

| **Field**  | **Type** | **Required** | **Description**                                                                        |
|------------|----------|:------------:|----------------------------------------------------------------------------------------|
| `limit`    | Number   |     Yes      | Maximum value allowed within the given period of time (duration)                       |
| `window`   | String   |     Yes      | The period of time that the limit applies. Follows [Gateway API Duration format](https://gateway-api.sigs.k8s.io/geps/gep-2257/?h=duration#gateway-api-duration-format) |

## RateLimitPolicyStatus

| **Field**            | **Type**                          | **Description**                                                                                                                     |
|----------------------|-----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------|
| `observedGeneration` | String                            | Number of the last observed generation of the resource. Use it to check if the status info is up to date with latest resource spec. |
| `conditions`         | [][ConditionSpec](#conditionspec) | List of conditions that define that status of the resource.                                                                         |

## ConditionSpec

* The *lastTransitionTime* field provides a timestamp for when the entity last transitioned from one status to another.
* The *message* field is a human-readable message indicating details about the transition.
* The *reason* field is a unique, one-word, CamelCase reason for the condition’s last transition.
* The *status* field is a string, with possible values **True**, **False**, and **Unknown**.
* The *type* field is a string with the following possible values:
    * Available: the resource has successfully configured;

| **Field**            | **Type**  | **Description**              |
|----------------------|-----------|------------------------------|
| `type`               | String    | Condition Type               |
| `status`             | String    | Status: True, False, Unknown |
| `reason`             | String    | Condition state reason       |
| `message`            | String    | Condition state description  |
| `lastTransitionTime` | Timestamp | Last transition timestamp    |

#### High-level example

```yaml
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: my-rate-limit-policy
spec:
  # Reference to an existing networking resource to attach the policy to. REQUIRED.
  # It can be a Gateway API HTTPRoute, GRPCRoute, or Gateway resource.
  # It can only refer to objects in the same namespace as the RateLimitPolicy.
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute # or GRPCRoute, Gateway
    name: myroute

  # The limits definitions to apply to the network traffic routed through the targeted resource.
  # Equivalent to if otherwise declared within `defaults`.
  limits:
    "my_limit":
      # The rate limits associated with this limit definition. REQUIRED.
      # E.g., to specify a 50rps rate limit, add `{ limit: 50, duration: 1, unit: secod }`
      rates: […]

      # Counter qualifiers.
      # Each dynamic value in the data plane starts a separate counter, combined with each rate limit.
      # E.g., to define a separate rate limit for each user name detected by the auth layer, add `metadata.filter_metadata.envoy\.filters\.http\.ext_authz.username`.
      # Check out Kuadrant RFC 0002 (https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md) to learn more about the Well-known Attributes that can be used in this field.
      counters: […]

      # Additional dynamic conditions to trigger the limit.
      # Use it for filtering attributes not supported by route rules or with RateLimitPolicies that target a Gateway.
      # Check out Kuadrant RFC 0002 (https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md) to learn more about the Well-known Attributes that can be used in this field.
      when: […]

    # Explicit defaults. Used in policies that target a Gateway object to express default rules to be enforced on
    # routes that lack a more specific policy attached to.
    # Mutually exclusive with `overrides` and with declaring `limits` at the top-level of the spec.
    defaults:
      limits: { … }

    # Overrides. Used in policies that target a Gateway object to be enforced on all routes linked to the gateway,
    # thus also overriding any more specific policy occasionally attached to any of those routes.
    # Mutually exclusive with `defaults` and with declaring `limits` at the top-level of the spec.
    overrides:
      limits: { … }
```

=== UNIT ID: kuadrant-operator/doc/reference/tokenratelimitpolicy.md:tokenratelimitpolicyspec ===
DOC: kuadrant-operator/doc/reference/tokenratelimitpolicy.md
TITLE: TokenRateLimitPolicySpec
BREADCRUMB: ['The TokenRateLimitPolicy Custom Resource Definition (CRD)']
TOKENS: 1884
## TokenRateLimitPolicy

| **Field** | **Type**                                        | **Required** | **Description**                                       |
|-----------|-------------------------------------------------|:------------:|-------------------------------------------------------|
| `spec`    | [TokenRateLimitPolicySpec](#tokenratelimitpolicyspec)     |     Yes      | The specification for TokenRateLimitPolicy custom resource |
| `status`  | [TokenRateLimitPolicyStatus](#tokenratelimitpolicystatus) |      No      | The status for the custom resource                    |

| **Field**   | **Type**                                                                                                                                    | **Required** | **Description**                                                                                                                                                                             |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------|--------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `targetRef` | [LocalPolicyTargetReferenceWithSectionName](#localpolicytargetreferencewithsectionname) | Yes          | Reference to a Kubernetes resource that the policy attaches to. For more [info](https://gateway-api.sigs.k8s.io/reference/spec/#localpolicytargetreferencewithsectionname)                                                                                                                              |
| `defaults`  | [MergeableTokenRateLimitPolicySpec](#mergeabletokenratelimitpolicyspec)                                                                                     | No           | Default limit definitions. This field is mutually exclusive with the `limits` field                                                                                                         |
| `overrides` | [MergeableTokenRateLimitPolicySpec](#mergeabletokenratelimitpolicyspec)                                                                                     | No           | Overrides limit definitions. This field is mutually exclusive with the `limits` field and `defaults` field. This field is only allowed for policies targeting `Gateway` in `targetRef.kind` |
| `limits`    | Map<String: [TokenLimit](#tokenlimit)>                                                                                                                | No           | Limit definitions. This field is mutually exclusive with the [`defaults`](#mergeabletokenratelimitpolicyspec) field                                                                                 |

## LocalPolicyTargetReferenceWithSectionName

| **Field**       | **Type**                                | **Required** | **Description**                                            |
|------------------|-----------------------------------------|--------------|------------------------------------------------------------|
| `LocalPolicyTargetReference`         | [LocalPolicyTargetReference](#localpolicytargetreference)          | Yes          | Reference to a local policy target.               |
| `sectionName`    | [SectionName](#sectionname)                         | No           | Section name for further specificity (if needed). |

## LocalPolicyTargetReference

| **Field** | **Type**     | **Required** | **Description**                |
|-----------|--------------|--------------|--------------------------------|
| `group`   | `Group`      | Yes          | Group of the target resource. |
| `kind`    | `Kind`       | Yes          | Kind of the target resource.  |
| `name`    | `ObjectName` | Yes          | Name of the target resource.  |

## SectionName

| Field       | Type                     | Required | Description                                                                                                                                                                                                                         |
|-------------|--------------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SectionName | v1.SectionName (String)  | Yes      | SectionName is the name of a section in a Kubernetes resource. <br>In the following resources, SectionName is interpreted as the following: <br>* Gateway: Listener name<br>* HTTPRoute: HTTPRouteRule name<br>* Service: Port name |

## MergeableTokenRateLimitPolicySpec

| **Field** | **Type**                     | **Required** | **Description**                                                                                                              |
|-----------|------------------------------|--------------|------------------------------------------------------------------------------------------------------------------------------|
| `strategy`| String                       | No           | Merge strategy to apply when merging with other policies. Values: `atomic` (default), `merge`                               |
| `limits`  | Map<String: [TokenLimit](#tokenlimit)> | Yes           | Map of named token-based rate limit configurations                                                                   |

## TokenLimit

| **Field** | **Type**                     | **Required** | **Description**                                                                                                              |
|-----------|------------------------------|--------------|------------------------------------------------------------------------------------------------------------------------------|
| `rates`   | [][Rate](#rate)              | No           | List of rate limit details including limit and window. If not specified, no rate limits are applied for this limit definition |
| `when`    | [][WhenPredicate](#whenpredicate)    | No           | List of predicates for this limit. Used in combination with top-level predicates                                     |
| `counters`| [][Counter](#counter)        | No           | CEL expressions that define counter keys for rate limiting. If not specified, rate limiting will be applied globally without user-specific tracking |

## Rate

| **Field** | **Type** | **Required** | **Description**                                                |
|-----------|----------|--------------|----------------------------------------------------------------|
| `limit`   | Number   | Yes          | Maximum token count allowed for the given window               |
| `window`  | Duration | Yes          | Time window for the limit (e.g., "1h", "24h", "1m", "1d")    |

## WhenPredicate

| **Field**   | **Type** | **Required** | **Description**                                                                                          |
|-------------|----------|--------------|----------------------------------------------------------------------------------------------------------|
| `predicate` | String   | Yes          | CEL expression that must evaluate to true for the limit to apply. See [Well-known Attributes](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md) |

=== UNIT ID: kuadrant-operator/doc/reference/tokenratelimitpolicy.md:counter ===
DOC: kuadrant-operator/doc/reference/tokenratelimitpolicy.md
TITLE: Counter
BREADCRUMB: ['The TokenRateLimitPolicy Custom Resource Definition (CRD)', 'TokenRateLimitPolicySpec']
TOKENS: 1715
| **Field**    | **Type** | **Required** | **Description**                                                                                         |
|--------------|----------|--------------|--------------------------------------------------------------------------------------------------------|
| `expression` | String   | Yes          | CEL expression that defines the counter key for rate limiting. See [Well-known Attributes](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md) |

## TokenRateLimitPolicyStatus

The status object for TokenRateLimitPolicy follows the [PolicyStatus](https://gateway-api.sigs.k8s.io/reference/spec/#gateway.networking.k8s.io/v1alpha2.PolicyStatus) pattern from Gateway API.

| **Field**      | **Type**                              | **Description**                                           |
|----------------|---------------------------------------|-----------------------------------------------------------|
| `observedGeneration` | Number                          | Generation of the resource that was last reconciled      |
| `conditions`   | [][Condition](#condition)             | Current state of the policy                              |

## Condition

Standard Kubernetes condition fields following Gateway API conventions:

| **Field**         | **Type**      | **Description**                                                    |
|-------------------|---------------|---------------------------------------------------------------------|
| `type`            | String        | Type of condition (e.g., "Accepted", "Enforced")                  |
| `status`          | String        | Status of the condition ("True", "False", "Unknown")              |
| `observedGeneration` | Number     | Generation observed when this condition was last updated           |
| `lastTransitionTime` | Timestamp  | Last time the condition transitioned from one status to another    |
| `reason`          | String        | Machine-readable reason for the condition's last transition        |
| `message`         | String        | Human-readable message indicating details about the last transition |

## Token Usage Tracking

TokenRateLimitPolicy automatically tracks token consumption from AI/LLM responses by monitoring the `usage.total_tokens` field in response bodies. This enables accurate usage-based rate limiting where:

- **Request Phase**: The policy evaluates predicates and descriptors during the request
- **Response Phase**: The policy extracts actual token usage from the response body
- **Rate Limiting**: Limitador receives the actual token count as `hits_addend` for precise accounting

## Supported Response Format

The policy automatically parses token usage from response bodies in the following format:
```json
{
  "usage": {
    "total_tokens": 150,
    "prompt_tokens": 100,
    "completion_tokens": 50
  }
}
```

This is compatible with OpenAI-style API responses and similar AI/LLM services.

**What's actually checked**: Token extraction looks for a single JSON pointer, `/usage/total_tokens`, in the response body. Any backend that returns that exact field works out of the box, including OpenAI Chat Completions (`/v1/chat/completions`), OpenAI legacy Completions (`/v1/completions`), OpenAI Embeddings, and OpenAI-compatible backends like vLLM, kServe, Ollama, Azure OpenAI, and Gemini's OpenAI-compatibility endpoint.

**Not currently supported**: Providers that return token usage under a different field name — Anthropic's `/v1/messages` (`usage.input_tokens` / `usage.output_tokens`, no `total_tokens`) or Google Gemini's native endpoint (`usageMetadata.totalTokenCount`) — are not parsed. Multi-provider support is tracked in [#1864](https://github.com/Kuadrant/kuadrant-operator/issues/1864).

**Streaming Support**: Both streaming and non-streaming responses are supported:
- **Non-streaming**: Works with `stream: false` or when `stream` is omitted
- **Streaming**: Requires `"stream": true` and `"stream_options": { "include_usage": true }` to extract usage from the final stream event

## CEL Expression Context

TokenRateLimitPolicy provides access to request attributes through CEL expressions. For a comprehensive list of available attributes, see the [Well-known Attributes RFC](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md).

Common attributes include:

| Context | Available Attributes | Example Usage |
|---------|---------------------|---------------|
| **Request** | `request.method`, `request.url_path`, `request.headers` | `request.method == "POST"` |
| **Authentication** | `auth.identity.*`, `request.auth.claims.*` | `auth.identity.userid`, `request.auth.claims["tier"]` |
| **Remote Address** | `source.address`, `source.port` | `source.address` |

## Basic Token Rate Limiting

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: basic-token-limit
  namespace: gateway-system
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ai-gateway
  limits:
    global:
      rates:
      - limit: 100000
        window: 1h
```

## User-Based Token Limiting

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: user-token-limits
  namespace: gateway-system
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: api-gateway
  limits:
    free:
      rates:
      - limit: 50000
        window: 24h
      when:
      - predicate: request.path == "/v1/chat/completions"
      - predicate: 'auth.identity.groups.split(",").exists(g, g == "free")'
      counters:
      - expression: auth.identity.userid
    gold:
      rates:
      - limit: 200000
        window: 24h
      when:
      - predicate: request.path == "/v1/chat/completions"
      - predicate: 'auth.identity.groups.split(",").exists(g, g == "gold")'
      counters:
      - expression: auth.identity.userid
```

## Gateway Overrides

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: org-wide-limits
  namespace: gateway-system
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: api-gateway
  overrides:
    strategy: atomic
    limits:
      org-quota:
        rates:
        - limit: 1000000
          window: 24h
        counters:
        - expression: auth.identity.org_id
```

## See Also

- [TokenRateLimitPolicy Overview](../overviews/token-rate-limiting.md)
- [Token Rate Limiting Tutorial](../user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md)
- [RateLimitPolicy Reference](ratelimitpolicy.md)
- [AuthPolicy Reference](authpolicy.md)
- [Well-known Attributes](https://github.com/Kuadrant/architecture/blob/main/rfcs/0002-well-known-attributes.md)
- [Gateway API Documentation](https://gateway-api.sigs.k8s.io/)

=== UNIT ID: kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md:set-the-environment-variables ===
DOC: kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md
TITLE: Set the environment variables
BREADCRUMB: ['Secure, protect, and connect APIs with Kuadrant', 'Prerequisites']
TOKENS: 1862
## Prerequisites

## Overview

This tutorial walks you through using Kuadrant to secure, protect, and connect an API exposed by a Gateway (Kubernetes Gateway API) from the personas platform engineer and application developer. For more information on the different personas please see the [Gateway API documentation](https://gateway-api.sigs.k8s.io/concepts/roles-and-personas/#key-roles-and-personas)

- Kubernetes cluster with Kuadrant operator installed. See our [Getting Started](/latest/getting-started) guide for more information.
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) command line tool.
- AWS/Azure or GCP with DNS capabilities.

Set the following environment variables used for convenience in this tutorial:

```bash
export KUADRANT_GATEWAY_NS=api-gateway # Namespace for the example Gateway
export KUADRANT_GATEWAY_NAME=external # Name for the example Gateway
export KUADRANT_DEVELOPER_NS=toystore # Namespace for an example toystore app
export KUADRANT_AWS_ACCESS_KEY_ID=xxxx # AWS Key ID with access to manage the DNS Zone ID below
export KUADRANT_AWS_SECRET_ACCESS_KEY=xxxx # AWS Secret Access Key with access to manage the DNS Zone ID below
export KUADRANT_AWS_DNS_PUBLIC_ZONE_ID=xxxx # AWS Route 53 Zone ID for the Gateway
export KUADRANT_ZONE_ROOT_DOMAIN=example.com # Root domain associated with the Zone ID above
export KUADRANT_CLUSTER_ISSUER_NAME=self-signed # Name for the ClusterIssuer
```

## Set up a DNS Provider

The DNS provider declares credentials to access the zone(s) that Kuadrant can use to set up DNS configuration. Ensure that this credential only has access to the zones you want Kuadrant to manage via `DNSPolicy`

Create the namespace the Gateway will be deployed in:

```bash
kubectl create ns ${KUADRANT_GATEWAY_NS}
```

Create the secret credentials in the same namespace as the Gateway - these will be used to configure DNS:

```bash
kubectl -n ${KUADRANT_GATEWAY_NS} create secret generic aws-credentials \
  --type=kuadrant.io/aws \
  --from-literal=AWS_ACCESS_KEY_ID=$KUADRANT_AWS_ACCESS_KEY_ID \
  --from-literal=AWS_SECRET_ACCESS_KEY=$KUADRANT_AWS_SECRET_ACCESS_KEY
```

Before adding a TLS issuer, create the secret credentials in the cert-manager namespace:

```bash
kubectl -n cert-manager create secret generic aws-credentials \
  --type=kuadrant.io/aws \
  --from-literal=AWS_ACCESS_KEY_ID=$KUADRANT_AWS_ACCESS_KEY_ID \
  --from-literal=AWS_SECRET_ACCESS_KEY=$KUADRANT_AWS_SECRET_ACCESS_KEY
```

## Deploy the Toystore app

Create the namespace for the Toystore application:

```bash

kubectl create ns ${KUADRANT_DEVELOPER_NS}
```

Deploy the Toystore app to the developer namespace:

```bash
kubectl apply -f https://raw.githubusercontent.com/Kuadrant/Kuadrant-operator/main/examples/toystore/toystore.yaml -n ${KUADRANT_DEVELOPER_NS}
```

## Add a TLS issuer

To secure communication to the Gateways, define a TLS issuer for TLS certificates.

!!! note 
    This example uses Let's Encrypt, but you can use any issuer supported by `cert-manager`.

```bash
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: ${KUADRANT_CLUSTER_ISSUER_NAME}
spec:
  selfSigned: {}
EOF
```

Wait for the `ClusterIssuer` to become ready.

```bash
kubectl wait clusterissuer/${KUADRANT_CLUSTER_ISSUER_NAME} --for=condition=ready=true
```

## Deploy a Gateway

```bash
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: ${KUADRANT_GATEWAY_NAME}
  namespace: ${KUADRANT_GATEWAY_NS}
  labels:
    kuadrant.io/gateway: "true"
spec:
    gatewayClassName: istio
    listeners:
    - allowedRoutes:
        namespaces:
          from: All 
      hostname: "api.${KUADRANT_ZONE_ROOT_DOMAIN}"
      name: api
      port: 443
      protocol: HTTPS
      tls:
        certificateRefs:
        - group: ""
          kind: Secret
          name: api-${KUADRANT_GATEWAY_NAME}-tls
        mode: Terminate
EOF
```

Check the status of the `Gateway` ensuring the gateway is Accepted and Programmed:

```bash
kubectl get gateway ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Programmed")].message}'
```

Check the status of the listener, you will see that it is not yet programmed or ready to accept traffic due to bad TLS configuration. This will be fixed in the next step with the `TLSPolicy`:

```bash
kubectl get gateway ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.listeners[0].conditions[?(@.type=="Programmed")].message}'
```

## Deploy the gateway TLS policy

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: TLSPolicy
metadata:
  name: ${KUADRANT_GATEWAY_NAME}-tls
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  targetRef:
    name: ${KUADRANT_GATEWAY_NAME}
    group: gateway.networking.k8s.io
    kind: Gateway
  issuerRef:
    group: cert-manager.io
    kind: ClusterIssuer
    name: ${KUADRANT_CLUSTER_ISSUER_NAME}
EOF
```

Check that the `TLSpolicy` has an Accepted and Enforced status (This may take a few minutes for certain provider e.g Lets Encrypt):


```bash
kubectl get tlspolicy ${KUADRANT_GATEWAY_NAME}-tls -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Enforced")].message}'
```

## Setup Toystore application HTTPRoute

```bash

kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: toystore
  namespace: ${KUADRANT_DEVELOPER_NS}
  labels:
    deployment: toystore
    service: toystore
spec:
  parentRefs:
  - name: ${KUADRANT_GATEWAY_NAME}
    namespace: ${KUADRANT_GATEWAY_NS}
  hostnames:
  - "api.${KUADRANT_ZONE_ROOT_DOMAIN}"
  rules:
  - matches:
    - method: GET
      path:
        type: PathPrefix
        value: "/cars"
    - method: GET
      path:
        type: PathPrefix
        value: "/health"    
    backendRefs:
    - name: toystore
      port: 80  
EOF
```

While the `Gateway` is now deployed, it currently has exposed endpoints. The next steps will be defining an `AuthPolicy` to set up a default `403` response for any unprotected endpoints, as well as a `RateLimitPolicy` to set up a default unrealistic low global limit to further protect any exposed endpoints.

## Set the `Deny all` Gateway AuthPolicy

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: ${KUADRANT_GATEWAY_NAME}-auth
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ${KUADRANT_GATEWAY_NAME}
  defaults:
   when:
     - predicate: "request.path != '/health'"
   rules:
    authorization:
      deny-all:
        opa:
          rego: "allow = false"
    response:
      unauthorized:
        headers:
          "content-type":
            value: application/json
        body:
          value: |
            {
              "error": "Forbidden",
              "message": "Access denied by default by the gateway operator. If you are the administrator of the service, create a specific auth policy for the route."
            }
EOF
```

Check that the `AuthPolicy` has Accepted and Enforced status:

```bash
kubectl get authpolicy ${KUADRANT_GATEWAY_NAME}-auth -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Enforced")].message}'

```

=== UNIT ID: kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md:deploy-the-low-limit-gateway-ratelimitpolicy ===
DOC: kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md
TITLE: Deploy the `low-limit` Gateway RateLimitPolicy
BREADCRUMB: ['Secure, protect, and connect APIs with Kuadrant', 'Secure and protect the Gateway with Auth, Rate Limit, and DNS policies.']
TOKENS: 1583
```bash
kubectl apply -f  - <<EOF
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: ${KUADRANT_GATEWAY_NAME}-rlp
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ${KUADRANT_GATEWAY_NAME}
  defaults:
    limits:
      "low-limit":
        rates:
        - limit: 1
          window: 10s
EOF
```

Check that the `RateLimitPolicy` has Accepted and Enforced status:

```bash
kubectl get ratelimitpolicy ${KUADRANT_GATEWAY_NAME}-rlp -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Enforced")].message}'
```

## Create the Gateway DNSPolicy

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: DNSPolicy
metadata:
  name: ${KUADRANT_GATEWAY_NAME}-dnspolicy
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  healthCheck:
    failureThreshold: 3
    interval: 1m
    path: /health
  loadBalancing:
    defaultGeo: true
    geo: GEO-NA
    weight: 120
  targetRef:
    name: ${KUADRANT_GATEWAY_NAME}
    group: gateway.networking.k8s.io
    kind: Gateway
  providerRefs:
  - name: aws-credentials # Secret created earlier
EOF
```

Check that the `DNSPolicy` has been Accepted and Enforced (This mat take a few minutes):

```bash
kubectl get dnspolicy ${KUADRANT_GATEWAY_NAME}-dnspolicy -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Enforced")].message}'
```

#### DNS Health checks
DNS Health checks has been enabled on the DNSPolicy. These health checks will flag a published endpoint as healthy or unhealthy based on the defined configuration. When unhealthy an endpoint will not be published if it has not already been published to the DNS provider, will only be unpublished if it is part of a multi-value A record and in all cases can be observable via the DNSPolicy status. For more information see [DNS Health checks documentation](../dns/dnshealthchecks.md)

Check the status of the health checks as follow:

```bash
kubectl get dnspolicy ${KUADRANT_GATEWAY_NAME}-dnspolicy -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="SubResourcesHealthy")].message}'
```

## Test the `low-limit` and `deny all` policies

```bash
while :; do curl -k --write-out '%{http_code}\n' --silent --output /dev/null  "https://api.$KUADRANT_ZONE_ROOT_DOMAIN/cars" | grep -E --color "\b(429)\b|$"; sleep 1; done
```

## Set up API key auth flow

Set up an example API key for the new users:

```bash
export KUADRANT_SYSTEM_NS=$(kubectl get kuadrant -A -o jsonpath="{.items[0].metadata.namespace}")
```

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: bob-key
  namespace: ${KUADRANT_SYSTEM_NS}
  labels:
    authorino.kuadrant.io/managed-by: authorino
    app: toystore
  annotations:
    secret.kuadrant.io/user-id: bob
stringData:
  api_key: IAMBOB
type: Opaque
---
apiVersion: v1
kind: Secret
metadata:
  name: alice-key
  namespace: ${KUADRANT_SYSTEM_NS}
  labels:
    authorino.kuadrant.io/managed-by: authorino
    app: toystore
  annotations:
    secret.kuadrant.io/user-id: alice
stringData:
  api_key: IAMALICE
type: Opaque
EOF
```

Create a new AuthPolicy in a different namespace that overrides the `Deny all` created earlier:



```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: toystore-auth
  namespace: ${KUADRANT_DEVELOPER_NS}
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: toystore
  defaults:
   when:
     - predicate: "request.path != '/health'"  
   rules:
    authentication:
      "api-key-users":
        apiKey:
          selector:
            matchLabels:
              app: toystore
        credentials:
          authorizationHeader:
            prefix: APIKEY
    response:
      success:
        filters:
          "identity":
            json:
              properties:
                "userid":
                  selector: auth.identity.metadata.annotations.secret\.kuadrant\.io/user-id
EOF
```

## Override `low-limit` RateLimitPolicy for specific users

Create a new `RateLimitPolicy` in a different namespace to override the default `RateLimitPolicy` created earlier:

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: toystore-rlp
  namespace: ${KUADRANT_DEVELOPER_NS}
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: toystore
  limits:
    "general-user":
      rates:
      - limit: 5
        window: 10s
      counters:
      - expression: auth.identity.userid
      when:
      - predicate: "auth.identity.userid != 'bob'"
    "bob-limit":
      rates:
      - limit: 2
        window: 10s
      when:
      - predicate: "auth.identity.userid == 'bob'"
EOF
```

The `RateLimitPolicy` should be Accepted and Enforced:

```bash
kubectl get ratelimitpolicy -n ${KUADRANT_DEVELOPER_NS} toystore-rlp -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Enforced")].message}'

```

Check the status of the `HTTPRoute`, is now affected by the `RateLimitPolicy` in the same namespace:

```bash
kubectl get httproute toystore -n ${KUADRANT_DEVELOPER_NS} -o=jsonpath='{.status.parents[0].conditions[?(@.type=="kuadrant.io/RateLimitPolicyAffected")].message}'
```

## Test the new Rate limit and Auth policy

#### Send requests as Alice:

You should see status `200` every second for 5 second followed by stats `429` every second for 5 seconds

```bash
while :; do curl -k --write-out '%{http_code}\n' --silent --output /dev/null -H 'Authorization: APIKEY IAMALICE' "https://api.$KUADRANT_ZONE_ROOT_DOMAIN/cars" | grep -E --color "\b(429)\b|$"; sleep 1; done
```

#### Send requests as Bob:
You should see status `200` every second for 2 seconds followed by stats `429` every second for 8 seconds


```bash
while :; do curl -k --write-out '%{http_code}\n' --silent --output /dev/null -H 'Authorization: APIKEY IAMBOB' "https://api.$KUADRANT_ZONE_ROOT_DOMAIN/cars" | grep -E --color "\b(429)\b|$"; sleep 1; done
```

## Next Steps

- [mTLS Configuration](../../install/mtls-configuration.md)
- [Configure Observability of Gateway and Kuadrant components](../../user-guides/observability/monitors.md)

=== UNIT ID: kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md:environment-variables ===
DOC: kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md
TITLE: Environment variables
BREADCRUMB: ['Token-based Rate Limiting for Large Language Model APIs']
TOKENS: 1954
## Prerequisites

## Overview

## Token-based Rate Limiting for Large Language Model APIs

This tutorial walks you through configuring token-based rate limiting using Kuadrant's `TokenRateLimitPolicy` to protect Large Language Model (LLM) APIs. Unlike traditional request counting, this approach limits API usage based on actual token consumption.

Traditional rate limiting counts requests, but LLM APIs have varying computational costs based on token usage. `TokenRateLimitPolicy` addresses this by:

- Counting actual tokens consumed from LLM responses
- Setting different limits for different user tiers when combined with `AuthPolicy`
- Integrating seamlessly with authentication policies

- Kubernetes cluster with Kuadrant operator installed. See our [Getting Started](/latest/getting-started) guide for more information.
- [kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl) command line tool.
- Basic understanding of [Gateway API](https://gateway-api.sigs.k8s.io/).

You should also have an instance of `Kuadrant` installed:

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1beta1
kind: Kuadrant
metadata:
  name: kuadrant
  namespace: kuadrant-system
EOF
```

Set the following environment variables used throughout this tutorial:

```bash
export KUADRANT_GATEWAY_NS=gateway-system
export KUADRANT_GATEWAY_NAME=trlp-tutorial-gateway
export KUADRANT_SYSTEM_NS=$(kubectl get kuadrant -A -o jsonpath='{.items[0].metadata.namespace}')
```

## Step 1: Deploy an LLM service

Deploy a simulated LLM service that mimics OpenAI-compatible APIs:

```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trlp-tutorial-llm-sim
  labels:
    app: trlp-tutorial-llm-sim
spec:
  replicas: 1
  selector:
    matchLabels:
      app: trlp-tutorial-llm-sim
  template:
    metadata:
      labels:
        app: trlp-tutorial-llm-sim
    spec:
      containers:
      - name: simulator
        image: ghcr.io/llm-d/llm-d-inference-sim:v0.1.1
        args:
          - --model
          - meta-llama/Llama-3.1-8B-Instruct
          - --port
          - "8000"
        ports:
          - containerPort: 8000
---
apiVersion: v1
kind: Service
metadata:
  name: trlp-tutorial-llm-sim
spec:
  selector:
    app: trlp-tutorial-llm-sim
  ports:
    - port: 80
      targetPort: 8000
      protocol: TCP
EOF
```

## Step 2: Create a Gateway

Create the namespace for the gateway:

```bash
kubectl create ns ${KUADRANT_GATEWAY_NS}
```

Create a gateway that will accept traffic for the LLM API:

```bash
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: ${KUADRANT_GATEWAY_NAME}
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  gatewayClassName: istio
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    hostname: "trlp-tutorial.example.com"
    allowedRoutes:
      namespaces:
        from: All
EOF
```

Check the gateway status:

```bash
kubectl get gateway ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Programmed")].message}{"\n"}'
```

## Step 3: Expose the service via HTTPRoute

Create an HTTPRoute to expose the LLM service:

```bash
kubectl apply -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: trlp-tutorial-llm-sim
spec:
  hostnames:
    - trlp-tutorial.example.com
  parentRefs:
    - name: ${KUADRANT_GATEWAY_NAME}
      namespace: ${KUADRANT_GATEWAY_NS}
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: "/"
      backendRefs:
        - name: trlp-tutorial-llm-sim
          port: 80
EOF
```

Export the gateway URL for use in requests:

```bash
export KUADRANT_INGRESS_HOST=$(kubectl get gtw ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o jsonpath='{.status.addresses[0].value}')
export KUADRANT_INGRESS_PORT=$(kubectl get gtw ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o jsonpath='{.spec.listeners[?(@.name=="http")].port}')
export KUADRANT_GATEWAY_URL=${KUADRANT_INGRESS_HOST}:${KUADRANT_INGRESS_PORT}
```

Test connectivity to the service:

```bash
curl -H 'Host: trlp-tutorial.example.com' http://$KUADRANT_GATEWAY_URL/v1/models -i
# HTTP/1.1 200 OK
```

> **Note**: If the command above fails to hit the service on your environment, try forwarding requests to the gateway and accessing over localhost:
>
> ```bash
> kubectl port-forward -n ${KUADRANT_GATEWAY_NS} service/${KUADRANT_GATEWAY_NAME}-istio 9080:80 >/dev/null 2>&1 &
> export KUADRANT_GATEWAY_URL=localhost:9080
> ```

## Step 4: Set up API key authentication

Create API keys for different user tiers. This example creates two tiers: "free" and "gold":

```bash
# Create a free tier user
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: trlp-tutorial-api-key-free-user-1
  namespace: ${KUADRANT_SYSTEM_NS}
  labels:
    authorino.kuadrant.io/managed-by: authorino
    app: my-llm
  annotations:
    kuadrant.io/groups: free
    secret.kuadrant.io/user-id: user-1
stringData:
  api_key: iamafreeuser
type: Opaque
EOF
```

```bash
# Create a gold tier user
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: trlp-tutorial-api-key-gold-user-1
  namespace: ${KUADRANT_SYSTEM_NS}
  labels:
    authorino.kuadrant.io/managed-by: authorino
    app: my-llm
  annotations:
    kuadrant.io/groups: gold
    secret.kuadrant.io/user-id: user-2
stringData:
  api_key: iamagolduser
type: Opaque
EOF
```

## Step 5: Configure authentication policy

Create an AuthPolicy that validates API keys and extracts user information:

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: trlp-tutorial-llm-api-keys
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ${KUADRANT_GATEWAY_NAME}
  rules:
    authentication:
      api-key-users:
        apiKey:
          selector:
            matchLabels:
              app: my-llm
        credentials:
          authorizationHeader:
            prefix: APIKEY
    response:
      success:
        filters:
          identity:
            json:
              properties:
                groups:
                  selector: auth.identity.metadata.annotations.kuadrant\.io/groups
                userid:
                  selector: auth.identity.metadata.annotations.secret\.kuadrant\.io/user-id
    authorization:
      allow-groups:
        opa:
          rego: |
            groups := split(object.get(input.auth.identity.metadata.annotations, "kuadrant.io/groups", ""), ",")
            allow { groups[_] == "free" }
            allow { groups[_] == "gold" }
EOF
```

## Step 6: Apply token rate limiting

Create a `TokenRateLimitPolicy` with different token limits for each tier:

```bash
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: trlp-tutorial-token-limits
  namespace: ${KUADRANT_GATEWAY_NS}
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ${KUADRANT_GATEWAY_NAME}
  limits:
    free:
      rates:
        - limit: 50 # 50 tokens per minute for free users (small for testing)
          window: 1m
      when:
        - predicate: request.path == "/v1/chat/completions"
        - predicate: |
            auth.identity.groups.split(",").exists(g, g == "free")
      counters:
        - expression: auth.identity.userid
    gold:
      rates:
        - limit: 200 # 200 tokens per minute for gold users (small for testing)
          window: 1m
      when:
        - predicate: request.path == "/v1/chat/completions"
        - predicate: |
            auth.identity.groups.split(",").exists(g, g == "gold")
      counters:
        - expression: auth.identity.userid
EOF
```

=== UNIT ID: kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md:test-with-a-free-user ===
DOC: kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md
TITLE: Test with a free user
BREADCRUMB: ['Token-based Rate Limiting for Large Language Model APIs', 'Step 7: Test the configuration']
TOKENS: 1365
Make a chat completion request. Note that `stream: false` is explicitly set to ensure a non-streaming response:

```bash
curl -H 'Host: trlp-tutorial.example.com' \
     -H 'Authorization: APIKEY iamafreeuser' \
     -H 'Content-Type: application/json' \
     -X POST http://$KUADRANT_GATEWAY_URL/v1/chat/completions \
     -d '{
           "model": "meta-llama/Llama-3.1-8B-Instruct",
           "messages": [
             { "role": "user", "content": "What is Kubernetes?" }
           ],
           "max_tokens": 100,
           "stream": false,
           "usage": true
         }'
```

The response includes token usage information:

```json
{
  "choices": [...],
  "usage": {
    "prompt_tokens": 5,
    "completion_tokens": 95,
    "total_tokens": 100
  }
}
```

> **Note:** The `TokenRateLimitPolicy` automatically extracts the `total_tokens` value from this response and counts it against the user's limit.

## Test streaming with a free user

Make a chat completion request with streaming enabled. Ensure the request includes `"stream": true` and `"stream_options": { "include_usage": true }` so that usage is emitted at the end of the stream and can be enforced by the policy:

```bash
curl -H 'Host: trlp-tutorial.example.com' \
     -H 'Authorization: APIKEY iamafreeuser' \
     -H 'Content-Type: application/json' \
     -X POST http://$KUADRANT_GATEWAY_URL/v1/chat/completions \
     -d '{
           "model": "meta-llama/Llama-3.1-8B-Instruct",
           "messages": [
             { "role": "user", "content": "What is Kubernetes?" }
           ],
           "max_tokens": 100,
           "stream": true,
           "stream_options": {
             "include_usage": true
           }
         }'
```

> **Note:** If `stream_options.include_usage` is omitted when `stream: true`, Kuadrant cannot extract token usage from the stream. Depending on the policy `failureMode`, the request may either be allowed without limiting or rejected.

## Test with a gold user

```bash
curl -H 'Host: trlp-tutorial.example.com' \
     -H 'Authorization: APIKEY iamagolduser' \
     -H 'Content-Type: application/json' \
     -X POST http://$KUADRANT_GATEWAY_URL/v1/chat/completions \
     -d '{
           "model": "meta-llama/Llama-3.1-8B-Instruct",
           "messages": [
             { "role": "user", "content": "Explain cloud native architecture" }
           ],
           "max_tokens": 200,
           "stream": false,
           "usage": true
         }'
```

## How it works

1. **Authentication**: The AuthPolicy validates API keys and enriches requests with user metadata
2. **Token Extraction**: `TokenRateLimitPolicy` automatically extracts `usage.total_tokens` from LLM responses
3. **Rate Limiting**: Tokens are counted against user-specific limits based on their tier
4. **Enforcement**: When limits are exceeded, requests are rejected with HTTP 429 (Too Many Requests)

## Understanding the policy

The `TokenRateLimitPolicy` uses several key concepts:

- **`rates`**: Define the token limits and time windows
- **`when`**: Conditions that determine when a limit applies (based on user groups)
- **`counters`**: Identify what to count (user ID in this case)
- **Token extraction**: Automatically reads `usage.total_tokens` from non-streaming JSON responses; for streaming requests, requires `"stream": true` and `"stream_options": { "include_usage": true }` to extract usage from the final event

## Monitoring usage

Check the generated rate limit configuration:

```bash
# View the generated WasmPlugin configuration
kubectl get wasmplugin -n ${KUADRANT_GATEWAY_NS} kuadrant-${KUADRANT_GATEWAY_NAME} -o yaml
```

## Next steps

- Experiment with different token limits and time windows
- Add more user tiers with different limits
- Integrate with your actual LLM service

## Troubleshooting

If requests are being rejected unexpectedly:
1. Verify the API key is correct
2. Check if the user has exceeded their token limit
3. Ensure the LLM response includes `usage.total_tokens`
5. Review the `AuthPolicy` and `TokenRateLimitPolicy` status

```bash
# Check AuthPolicy status
kubectl get authpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-llm-api-keys -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}'

# Check TokenRateLimitPolicy status
kubectl get tokenratelimitpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-token-limits -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}'

# View full policy details if needed
kubectl get authpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-llm-api-keys -o yaml
kubectl get tokenratelimitpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-token-limits -o yaml
```

## Cleanup

To remove all resources created in this tutorial:

```bash
# Delete policies
kubectl delete tokenratelimitpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-token-limits
kubectl delete authpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-llm-api-keys

# Delete API key secrets
kubectl delete secret -n ${KUADRANT_SYSTEM_NS} trlp-tutorial-api-key-free-user-1 trlp-tutorial-api-key-gold-user-1

# Delete HTTPRoute
kubectl delete httproute trlp-tutorial-llm-sim

# Delete Gateway
kubectl delete gateway -n ${KUADRANT_GATEWAY_NS} ${KUADRANT_GATEWAY_NAME}

# Delete LLM service and deployment
kubectl delete service trlp-tutorial-llm-sim
kubectl delete deployment trlp-tutorial-llm-sim

# Delete the gateway namespace (if not used by other resources)
# kubectl delete namespace ${KUADRANT_GATEWAY_NS}
```

=== UNIT ID: kuadrant-operator/examples/toystore/admin-key-secret.yaml.md:admin-key-secretyaml ===
DOC: kuadrant-operator/examples/toystore/admin-key-secret.yaml.md
TITLE: admin-key-secret.yaml
BREADCRUMB: []
TOKENS: 82
## admin-key-secret.yaml

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  annotations:
    secret.kuadrant.io/user-id: alice
  creationTimestamp: null
  labels:
    app: toystore
    admin: "yes"
    authorino.kuadrant.io/managed-by: authorino
  name: toystore-admin-apikey
stringData:
  api_key: IAMADMIN
type: Opaque
```

=== UNIT ID: kuadrant-operator/examples/toystore/alice-api-key-secret.yaml.md:alice-api-key-secretyaml ===
DOC: kuadrant-operator/examples/toystore/alice-api-key-secret.yaml.md
TITLE: alice-api-key-secret.yaml
BREADCRUMB: []
TOKENS: 81
## alice-api-key-secret.yaml

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  annotations:
    secret.kuadrant.io/user-id: alice
  creationTimestamp: null
  labels:
    app: toystore
    authorino.kuadrant.io/managed-by: authorino
  name: toystore-alice-apikey
stringData:
  api_key: ALICEKEYFORDEMO
type: Opaque
```

=== UNIT ID: kuadrant-operator/examples/toystore/authpolicy.yaml.md:authpolicyyaml ===
DOC: kuadrant-operator/examples/toystore/authpolicy.yaml.md
TITLE: authpolicy.yaml
BREADCRUMB: []
TOKENS: 274
## authpolicy.yaml

```yaml
---
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: toystore
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: toystore
  rules:
    authentication:
      "apikey":
        apiKey:
          selector:
            matchLabels:
              app: toystore
      credentials:
        authorizationHeader:
          prefix: APIKEY
    response:
      success:
        filters:
          "ext_auth_data":
            json:
              properties:
                "user-id":
                  selector: auth.identity.metadata.annotations.secret\.kuadrant\.io/user-id
---
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: toystore
  namespace: gateway-system
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: kuadrant-ingressgateway
  rules:
    authentication:
      "apikey":
        apiKey:
          selector:
            matchLabels:
              app: toystore
              admin: "yes"
        credentials:
          authorizationHeader:
            prefix: APIKEY
```

=== UNIT ID: kuadrant-operator/examples/toystore/bob-api-key-secret.yaml.md:bob-api-key-secretyaml ===
DOC: kuadrant-operator/examples/toystore/bob-api-key-secret.yaml.md
TITLE: bob-api-key-secret.yaml
BREADCRUMB: []
TOKENS: 79
## bob-api-key-secret.yaml

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  annotations:
    secret.kuadrant.io/user-id: bob
  creationTimestamp: null
  labels:
    app: toystore
    authorino.kuadrant.io/managed-by: authorino
  name: toystore-bob-apikey
stringData:
  api_key: BOBKEYFORDEMO
type: Opaque
```

=== UNIT ID: kuadrant-operator/examples/toystore/httproute.yaml.md:httprouteyaml ===
DOC: kuadrant-operator/examples/toystore/httproute.yaml.md
TITLE: httproute.yaml
BREADCRUMB: []
TOKENS: 160
## httproute.yaml

```yaml
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: toystore
  labels:
    app: toystore
spec:
  parentRefs:
    - name: kuadrant-ingressgateway
      namespace: gateway-system
  hostnames: ["*.toystore.com"]
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: "/toy"
          method: GET
        - path:
            type: Exact
            value: "/admin/toy"
          method: POST
        - path:
            type: Exact
            value: "/admin/toy"
          method: DELETE
      backendRefs:
        - name: toystore
          port: 80
```

=== UNIT ID: kuadrant-operator/examples/toystore/ratelimitpolicy_gateway.yaml.md:ratelimitpolicy-gatewayyaml ===
DOC: kuadrant-operator/examples/toystore/ratelimitpolicy_gateway.yaml.md
TITLE: ratelimitpolicy_gateway.yaml
BREADCRUMB: []
TOKENS: 135
## ratelimitpolicy_gateway.yaml

```yaml
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: toystore-gw
  namespace: gateway-system
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: kuadrant-ingressgateway
  limits:
    "expensive-operation":
      rates:
      - limit: 2
        window: 30s
      when:
      - predicate: "request.method == 'POST'"

    "limit-per-ip":
      rates:
      - limit: 5
        window: 30s
      when:
      - predicate: "source.id == source.address"
```

=== UNIT ID: kuadrant-operator/examples/toystore/ratelimitpolicy_httproute.yaml.md:ratelimitpolicy-httprouteyaml ===
DOC: kuadrant-operator/examples/toystore/ratelimitpolicy_httproute.yaml.md
TITLE: ratelimitpolicy_httproute.yaml
BREADCRUMB: []
TOKENS: 268
## ratelimitpolicy_httproute.yaml

```yaml
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: toystore-httproute
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: toystore
  limits:
    "global":
      rates:
      - limit: 6
        window: 30s
    "get-toy":
      when:
      - predicate: "request.method == 'GET'"
      - predicate: "request.path == '/toy'"
      rates:
      - limit: 5
        window: 1m
    "admin-post-toy-per-user":
      rates:
      - limit: 2
        window: 30s
      counters:
      - expression: "auth.identity.username"
      when:
      - predicate: "request.method == 'GET'"
      - predicate: "request.path == '/admin/toy'"
      - predicate: "auth.identity.group == 'admin'"
    "admin-delete-per-user":
      rates:
      - limit: 2
        window: 30s
      counters:
      - expression: "auth.identity.username"
      when:
      - predicate: "request.method == 'DELETE'"
      - predicate: "request.path == '/admin/toy'"
      - predicate: "auth.identity.group == 'admin'"
```

=== UNIT ID: kuadrant-operator/examples/toystore/tokenratelimitpolicy_httproute.yaml.md:tokenratelimitpolicy-httprouteyaml ===
DOC: kuadrant-operator/examples/toystore/tokenratelimitpolicy_httproute.yaml.md
TITLE: tokenratelimitpolicy_httproute.yaml
BREADCRUMB: []
TOKENS: 121
## tokenratelimitpolicy_httproute.yaml

```yaml
apiVersion: kuadrant.io/v1alpha1
kind: TokenRateLimitPolicy
metadata:
  name: token-limit-free
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: toystore
  limits:
    free:
      rates:
      - limit: 20000
        window: 1d
      when:
      - predicate: 'request.auth.claims["kuadrant.io/groups"].split(",").exists(g, g == "free")'
      counters:
      - expression: auth.identity.userid
```

=== UNIT ID: kuadrant-operator/examples/toystore/toystore.yaml.md:toystoreyaml ===
DOC: kuadrant-operator/examples/toystore/toystore.yaml.md
TITLE: toystore.yaml
BREADCRUMB: []
TOKENS: 190
## toystore.yaml

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: toystore
  labels:
    app: toystore
spec:
  selector:
    matchLabels:
      app: toystore
  template:
    metadata:
      labels:
        app: toystore
    spec:
      containers:
        - name: toystore
          image: quay.io/kuadrant/authorino-examples:talker-api
          env:
            - name: LOG_LEVEL
              value: "debug"
            - name: PORT
              value: "3000"
          ports:
            - containerPort: 3000
              name: http
  replicas: 1
---
apiVersion: v1
kind: Service
metadata:
  name: toystore
spec:
  selector:
    app: toystore
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 3000
```

