---
type: procedure
products: [kuadrant]
versions: []
sources:
  - raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L14-470
verified: 2026-08-26
confidence: documented
---

# Token-based rate limiting for LLM APIs (tutorial)

Upstream procedure standing up [[kuadrant-tokenratelimitpolicy]] against a simulated OpenAI-compatible LLM service, combined with [[kuadrant-authpolicy]] for tiered (free/gold) user segmentation. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L14-25)

## Preconditions

- A Kubernetes cluster with the Kuadrant operator installed.
- `kubectl`.
- Basic Gateway API familiarity.
- A [[kuadrant-cr]] instance:
  ```bash
  kubectl apply -f - <<EOF
  apiVersion: kuadrant.io/v1beta1
  kind: Kuadrant
  metadata:
    name: kuadrant
    namespace: kuadrant-system
  EOF
  ```

(raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L26-43)

Environment variables:

```bash
export KUADRANT_GATEWAY_NS=gateway-system
export KUADRANT_GATEWAY_NAME=trlp-tutorial-gateway
export KUADRANT_SYSTEM_NS=$(kubectl get kuadrant -A -o jsonpath='{.items[0].metadata.namespace}')
```

(raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L44-49)

## Steps

1. **Deploy a simulated LLM service** — a `Deployment`/`Service` running `ghcr.io/llm-d/llm-d-inference-sim:v0.1.1` (`--model meta-llama/Llama-3.1-8B-Instruct --port 8000`), exposed on service port 80 → container port 8000. This stands in for a real OpenAI-compatible backend (e.g. vLLM/kServe in a real deployment). (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L54-100)

2. **Create the Gateway** — namespace `${KUADRANT_GATEWAY_NS}`, an HTTP listener on port 80, hostname `trlp-tutorial.example.com`. Verify `Accepted`/`Programmed`. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L101-136)

3. **Expose via HTTPRoute** — routes all paths on `trlp-tutorial.example.com` to the LLM sim service. Export the gateway URL:
   ```bash
   export KUADRANT_INGRESS_HOST=$(kubectl get gtw ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o jsonpath='{.status.addresses[0].value}')
   export KUADRANT_INGRESS_PORT=$(kubectl get gtw ${KUADRANT_GATEWAY_NAME} -n ${KUADRANT_GATEWAY_NS} -o jsonpath='{.spec.listeners[?(@.name=="http")].port}')
   export KUADRANT_GATEWAY_URL=${KUADRANT_INGRESS_HOST}:${KUADRANT_INGRESS_PORT}
   ```
   Verify with `curl -H 'Host: trlp-tutorial.example.com' http://$KUADRANT_GATEWAY_URL/v1/models -i` → expect `200 OK`. If that fails in your environment, `kubectl port-forward` the gateway's Istio service to `localhost:9080` instead. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L137-185)

4. **API-key secrets for two tiers** — free-tier secret (`trlp-tutorial-api-key-free-user-1`, annotated `kuadrant.io/groups: free`, `secret.kuadrant.io/user-id: user-1`, key `iamafreeuser`) and gold-tier secret (`trlp-tutorial-api-key-gold-user-1`, annotated `kuadrant.io/groups: gold`, `secret.kuadrant.io/user-id: user-2`, key `iamagolduser`), both created in `${KUADRANT_SYSTEM_NS}` (the Kuadrant CR's namespace) and labeled `authorino.kuadrant.io/managed-by: authorino, app: my-llm`. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L186-229)

5. **AuthPolicy** — targets the Gateway; authenticates via API key (`selector.matchLabels.app: my-llm`); an OPA authorization rule allows only `free` or `gold` groups; `response.success.filters` exposes `groups` (from the `kuadrant.io/groups` annotation) and `userid` (from `secret.kuadrant.io/user-id`) for downstream consumption. This is the exact fragment referenced from the [[kuadrant-tokenratelimitpolicy]] reference doc's "Integration with AuthPolicy" section. Verify `Accepted`. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L230-275)

6. **TokenRateLimitPolicy** — targets the same Gateway with two limits: `free` (50 tokens/min — deliberately small for testing) and `gold` (200 tokens/min), each gated on `request.path == "/v1/chat/completions"` and a groups-claim CEL predicate, counted per `auth.identity.userid`. Verify `Accepted`. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L276-315)

7. **Test as a free user (non-streaming)** — `POST /v1/chat/completions` with `Authorization: APIKEY iamafreeuser`, explicit `"stream": false`. Response includes `usage.total_tokens`, which the policy extracts and counts against the free-tier limit. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L316-344)

8. **Test streaming as a free user** — same request but `"stream": true` and `"stream_options": {"include_usage": true}` — required for usage to be emitted at the end of the stream and enforced; omitting `stream_options.include_usage` means Kuadrant can't extract usage, and the request is either allowed unmetered or rejected depending on the policy's `failureMode`. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L353-376)

9. **Test as a gold user** — same shape, `Authorization: APIKEY iamagolduser`, exercising the higher 200 tokens/min limit. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L377-394)

## Verification

- `kubectl get wasmplugin -n ${KUADRANT_GATEWAY_NS} kuadrant-${KUADRANT_GATEWAY_NAME} -o yaml` — inspect the generated data-plane config. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L411-419)
- `kubectl get authpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-llm-api-keys -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}'`
- `kubectl get tokenratelimitpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-token-limits -o=jsonpath='{.status.conditions[?(@.type=="Accepted")].message}'`

(raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L426-445)

## Troubleshooting

If requests are rejected unexpectedly: verify the API key is correct, check whether the user has exceeded their token limit, confirm the LLM response actually includes `usage.total_tokens`, and review AuthPolicy/TokenRateLimitPolicy status. (raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L426-444)

## Rollback / cleanup

```bash
kubectl delete tokenratelimitpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-token-limits
kubectl delete authpolicy -n ${KUADRANT_GATEWAY_NS} trlp-tutorial-llm-api-keys
kubectl delete secret -n ${KUADRANT_SYSTEM_NS} trlp-tutorial-api-key-free-user-1 trlp-tutorial-api-key-gold-user-1
kubectl delete httproute trlp-tutorial-llm-sim
kubectl delete gateway -n ${KUADRANT_GATEWAY_NS} ${KUADRANT_GATEWAY_NAME}
kubectl delete service trlp-tutorial-llm-sim
kubectl delete deployment trlp-tutorial-llm-sim
# kubectl delete namespace ${KUADRANT_GATEWAY_NS}  # only if unused by other resources
```

(raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md#L446-469)

## Related

- [[kuadrant-tokenratelimitpolicy]] — the CRD this tutorial exercises; full field reference.
- [[kuadrant-authpolicy]] — the tiered-auth pattern used at step 5 is the exact fragment cross-referenced from the reference doc's AuthPolicy-integration section.
- [[kuadrant-token-rate-limiting-overview]] — concept background: why token count instead of request count.
- [[kuadrant-secure-protect-connect-walkthrough]] — the sibling procedure covering request-count rate limiting and deny-all/API-key auth patterns.
- [[model-serving-platforms-overview]], [[kserve-model-serving-crds]] — the simulated LLM service here stands in for a real vLLM/kServe deployment; no RHOAI-specific version of this tutorial exists yet.
