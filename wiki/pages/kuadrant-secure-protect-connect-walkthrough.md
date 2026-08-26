---
type: procedure
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L14-486
  - raw/kuadrant-operator/examples/toystore/toystore.yaml.md#L1-56
verified: 2026-08-26
confidence: documented
---

# Secure, protect, and connect APIs with Kuadrant (full walkthrough)

End-to-end upstream procedure combining a Gateway, [[kuadrant-authpolicy]], and [[kuadrant-ratelimitpolicy]] — from the platform-engineer and app-developer personas, per the [Gateway API roles model](https://gateway-api.sigs.k8s.io/concepts/roles-and-personas/#key-roles-and-personas). (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L14-18)

## Preconditions

- A Kubernetes cluster with the Kuadrant operator installed (i.e. a [[kuadrant-cr]] instance) — see upstream Getting Started guide.
- `kubectl`.
- AWS, Azure, or GCP account with DNS capabilities (for the `DNSPolicy`/TLS steps — DNS/TLS are out of scope for this ingest's component pages, but the walkthrough exercises them as part of the full flow).

(raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L20-25)

Environment variables used throughout:

```bash
export KUADRANT_GATEWAY_NS=api-gateway
export KUADRANT_GATEWAY_NAME=external
export KUADRANT_DEVELOPER_NS=toystore
export KUADRANT_AWS_ACCESS_KEY_ID=xxxx
export KUADRANT_AWS_SECRET_ACCESS_KEY=xxxx
export KUADRANT_AWS_DNS_PUBLIC_ZONE_ID=xxxx
export KUADRANT_ZONE_ROOT_DOMAIN=example.com
export KUADRANT_CLUSTER_ISSUER_NAME=self-signed
```

(raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L26-40)

## Steps

1. **DNS provider secret** — create the gateway namespace and an `aws-credentials` secret (type `kuadrant.io/aws`) in it, plus a matching secret in the `cert-manager` namespace for the TLS issuer. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L41-67)

2. **Deploy the toystore app** — create the developer namespace and apply `examples/toystore/toystore.yaml` (a `Deployment` + `Service` running `quay.io/kuadrant/authorino-examples:talker-api` on port 3000, service port 80). (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L69-84, raw/kuadrant-operator/examples/toystore/toystore.yaml.md#L1-56)

3. **TLS issuer** — apply a `cert-manager.io/v1 ClusterIssuer` (self-signed in this example; any cert-manager-supported issuer works), then wait for it to become ready. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L85-108)

4. **Deploy the Gateway** — an HTTPS listener on `api.${KUADRANT_ZONE_ROOT_DOMAIN}` (port 443, `Terminate` TLS mode). Verify with:
   ```bash
   kubectl get gateway ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} \
     -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Programmed")].message}'
   ```
   Note the listener will report **not programmed** until the TLS policy (next step) is applied. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L109-150)

5. **Gateway `TLSPolicy`** — attaches the `ClusterIssuer` to the Gateway; verify `Accepted`/`Enforced` status (may take a few minutes for real issuers like Let's Encrypt). TLSPolicy detail is out of scope for this ingest's component pages. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L153-180)

6. **Toystore `HTTPRoute`** — routes `GET /cars` and `GET /health` on `api.${KUADRANT_ZONE_ROOT_DOMAIN}` to the toystore `Service`. At this point the endpoints are exposed but unprotected. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L181-217)

7. **Gateway-level "deny all" AuthPolicy** — a [[kuadrant-authpolicy]] targeting the Gateway with `defaults.rules.authorization` set to an OPA rule `allow = false`, plus a custom `403`-style JSON body via `response.unauthorized`, and a `when` predicate excluding `/health` from the deny-all. This establishes a secure-by-default baseline any route must explicitly open up. Verify `Accepted`/`Enforced`. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L218-260)

8. **Gateway-level `low-limit` RateLimitPolicy** — a [[kuadrant-ratelimitpolicy]] targeting the same Gateway with `defaults.limits."low-limit"` = 1 request/10s — an intentionally low default protecting any exposed endpoint. Verify `Accepted`/`Enforced`. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L261-279)

9. **Gateway `DNSPolicy`** — publishes the gateway address to Route 53, with a health check (`/health`, 3-failure threshold, 1m interval) and geo/weighted load-balancing config. DNSPolicy detail is out of scope for this ingest's component pages. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L289-329)

10. **Verify low-limit + deny-all in effect**:
    ```bash
    while :; do curl -k --write-out '%{http_code}\n' --silent --output /dev/null "https://api.$KUADRANT_ZONE_ROOT_DOMAIN/cars" | grep -E --color "\b(429)\b|$"; sleep 1; done
    ```
    (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L330-336)

11. **API-key secrets for two users** — `bob-key` and `alice-key`, created in the Kuadrant CR's namespace (`KUADRANT_SYSTEM_NS`, resolved via `kubectl get kuadrant -A`), each labeled `authorino.kuadrant.io/managed-by: authorino, app: toystore` and annotated `secret.kuadrant.io/user-id: <bob|alice>`. This exercises the [[kuadrant-cr]] namespace requirement documented on [[kuadrant-authpolicy]]. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L338-379)

12. **Route-level AuthPolicy override** — a new [[kuadrant-authpolicy]] in the developer namespace, targeting the `toystore` HTTPRoute, requiring API-key auth (label `app: toystore`) and exposing the resolved `user-id` annotation as `auth.identity.userid` via `response.success.filters`. This is a more specific policy that supersedes the Gateway-level deny-all default for this route. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L338-379)

13. **Route-level RateLimitPolicy override** — a new [[kuadrant-ratelimitpolicy]] in the developer namespace on the same route, with two limits: `general-user` (5 req/10s, counted by `auth.identity.userid`, excluding bob) and `bob-limit` (2 req/10s, specifically for bob). This overrides the Gateway-level `low-limit` default for this route. Verify `Accepted`/`Enforced` and that the HTTPRoute's `kuadrant.io/RateLimitPolicyAffected` condition reflects it. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L418-464)

14. **Verify per-user limits**:
    - As Alice (`Authorization: APIKEY IAMALICE`): `200` for 5s then `429` for 5s (matches `general-user`'s 5 req/10s).
    - As Bob (`Authorization: APIKEY IAMBOB`): `200` for 2s then `429` for 8s (matches `bob-limit`'s 2 req/10s).
    (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L465-482)

## Verification summary

Every applied CR in this walkthrough is checked the same way: `kubectl get <kind> <name> -n <ns> -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}{"\n"}{.status.conditions[?(@.type=="Enforced")].message}'` — expect both to resolve to a success message before moving on. (raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L131-279 — pattern repeated at each policy-creation step)

## Rollback

Not documented in this raw source — no `kubectl delete` cleanup sequence is given for this walkthrough (contrast with [[kuadrant-authenticated-token-rate-limiting-tutorial]], which does include one).

## Next steps (forward references, out of scope for this ingest)

- mTLS configuration
- Observability/monitoring of the Gateway and Kuadrant components

(raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md#L483-486)

## Related

- [[kuadrant-cr]], [[kuadrant-authpolicy]], [[kuadrant-ratelimitpolicy]] — the three CRDs this procedure exercises together.
- [[kuadrant-auth-overview]], [[kuadrant-rate-limiting-overview]] — concept background for the defaults/overrides pattern used at steps 7-8 and 12-13.
- [[kuadrant-authenticated-token-rate-limiting-tutorial]] — the sibling procedure for token-based (LLM) rate limiting, using a similar API-key + tiered-user pattern.
