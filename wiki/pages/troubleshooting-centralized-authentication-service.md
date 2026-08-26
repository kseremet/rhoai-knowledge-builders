---
type: error
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1790-1937
verified: 2026-08-25
confidence: documented
---

# Troubleshooting the centralized authentication service

Symptoms from setting up the [[centralized-authentication-service]] —
[[configuring-oidc-centralized-authentication]],
[[configuring-service-account-token-authentication]], and
[[configuring-custom-ca-for-oidc]]. If undocumented here or in the release
notes, contact Red Hat Support.

## Token duration error creating a service account token

**Symptom**: `"may not specify a duration less than 10 minutes"`.

**Cause**: Kubernetes enforces a 10-minute minimum token expiration; the
`--duration` flag was set lower.

**Fix**:
```
oc create token my-app-sa -n <your-namespace> --duration=10m
```

## HTTP 401 Unauthorized during Gateway access

**Symptom**: requests to the Gateway URL return `HTTP 401 Unauthorized`.

**Cause**: the bearer token is invalid — expired (past its `--duration`),
or `kube-auth-proxy` lacks the `TokenReview` permission needed to validate
it against the API server.

**Fix**:

1. Generate a new token with `oc create token` and a valid duration.
2. Confirm the header is `Authorization: Bearer ${TOKEN}`.
3. Check `kube-auth-proxy` logs in `openshift-ingress` for API-server
   connection errors.

## GatewayConfig status shows not ready

**Symptom**: `GatewayConfig` reports `Ready: False`, with errors about
missing OIDC configuration.

**Diagnosis**:
```
oc get gatewayconfig default-gateway -o yaml
oc describe gatewayconfig default-gateway
oc get gatewayconfig default-gateway -o jsonpath='{.spec.oidc}'
```
A successfully-provisioned config shows `Ready` and `ProvisioningSucceeded`
conditions both `True`, and the OIDC fields (`clientID`,
`clientSecretRef`, `issuerURL`) populated.

**Fix**:

1. Confirm the OIDC secret exists and is correct:
   `oc get secret keycloak-client-secret -n openshift-ingress`
2. Confirm the OIDC issuer URL is reachable:
   ```
   curl -I https://your-keycloak-domain/realms/your-realm/.well-known/openid-configuration
   ```
   Expect `HTTP/2 200` and `content-type: application/json`.
3. Confirm the client secret value is correct.

## Authentication proxy fails to start

**Symptom**: `kube-auth-proxy` pods in `CrashLoopBackOff` or `Pending`;
its Deployment never becomes ready.

**Diagnosis**:
```
oc get deployment kube-auth-proxy -n openshift-ingress
oc logs -l app=kube-auth-proxy -n openshift-ingress
oc describe pod -l app=kube-auth-proxy -n openshift-ingress
```
A healthy pod's logs show `"OAuth2 Proxy configured"` and
`"OAuth2 Proxy starting on :4180"` / `:8443`.

**Fix**:

1. Confirm the auth secret has the expected keys:
   ```
   oc get secret kube-auth-proxy-creds -n openshift-ingress -o yaml
   ```
   Expect `OAUTH2_PROXY_CLIENT_SECRET`, `OAUTH2_PROXY_COOKIE_SECRET`,
   `OAUTH2_PROXY_CLIENT_ID`.
2. Confirm the OIDC issuer URL is reachable from the cluster (same `curl`
   as above, expect `HTTP/2 200`).
3. Confirm the client ID exists in the OIDC provider.

## The Gateway is inaccessible

**Symptom**: `https://data-science-gateway.$CLUSTER_DOMAIN` returns 502
(Bad Gateway) or 503 (Service Unavailable).

**Diagnosis**:
```
oc get gateway data-science-gateway -n openshift-ingress
oc get httproute -n openshift-ingress
oc get envoyfilter -n openshift-ingress
oc get service kube-auth-proxy -n openshift-ingress
```
Expect: `PROGRAMMED: True` with a valid `ADDRESS`; `oauth-callback-route`
present in `HTTPRoute`; `authn-filter` present in `EnvoyFilter`; the
`kube-auth-proxy` Service with ports `8443/TCP,9000/TCP`.

**Fix**:

1. `oc get gateway data-science-gateway -n openshift-ingress -o jsonpath='{.status.addresses}'`
   — confirm a valid address.
2. `oc describe httproute oauth-callback-route -n openshift-ingress` —
   confirm proper parent references and backend services.
3. `oc describe envoyfilter authn-filter -n openshift-ingress` — confirm
   correct `kube-auth-proxy` configuration.

## OIDC authentication fails (redirect loops / auth errors)

**Diagnosis**:
```
oc logs -l app=kube-auth-proxy -n openshift-ingress
oc get secret kube-auth-proxy-creds -n openshift-ingress -o yaml
curl -s https://your-keycloak-domain/realms/your-realm/.well-known/openid-configuration | jq .
```
The Secret should contain `OAUTH2_PROXY_CLIENT_ID`,
`OAUTH2_PROXY_CLIENT_SECRET`, `OAUTH2_PROXY_COOKIE_SECRET`.

**Fix**:

1. In the OIDC provider (e.g. Keycloak), confirm the client's registered
   redirect URI matches
   `https://data-science-gateway.$CLUSTER_DOMAIN/oauth2/callback` —
   mismatches are the most common cause of redirect loops.
2. `echo $KEYCLOAK_CLIENT_SECRET | base64 -d` and compare against the
   provider's value.
3. `curl -I https://keycloak.example.com/realms/your-realm/.well-known/openid-configuration`
   — expect `HTTP/2 200`.

## Dashboard not accessible after authentication (403)

**Symptom**: authentication with OIDC succeeds, but the dashboard returns
403 Forbidden.

**Diagnosis**:
```
oc get deployment -n redhat-ods-applications rhods-dashboard
oc logs -l app=rhods-dashboard -n redhat-ods-applications
oc auth can-i get projects --as=your-username
```

**Fix**:

1. Grant cluster-level RBAC, e.g.:
   `oc adm policy add-cluster-role-to-user view your-username`
2. Confirm the `odh-dashboard` `HTTPRoute` has correct parent references
   to the Gateway:
   `oc get httproute rhods-dashboard -n redhat-ods-applications -o yaml`
3. Confirm the user is in the expected group (e.g. `odh-users`):
   `oc get user your-username -o yaml`

Distinct from [[rhoai-dashboard-not-accessible]], which covers the
generic post-install dashboard-unreachable case (not specific to
centralized/OIDC authentication).
