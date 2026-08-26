---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1675-1732
verified: 2026-08-25
confidence: documented
---

# Configuring OIDC for the centralized authentication service

Configures the [[centralized-authentication-service]]'s OIDC (interactive
browser) authentication path against an external identity provider (IdP).

**Important**: the OpenShift cluster itself must already be configured for
direct authentication with the **same** external OIDC provider *before*
configuring this service.

## Preconditions

- OpenShift cluster configured for direct authentication with an external
  OIDC provider — see OCP's "Enabling direct authentication with an
  external OIDC identity provider" (not ingested), or, on ROSA, "Creating
  an OpenID Connect configuration" (not ingested).
- RHOAI installed and deployed: [[datasciencecluster]] and
  [[dscinitialization]] both exist; the RHOAI Operator runs in the
  `rhods-operator` namespace.
- Gateway API support enabled on OCP 4.19.9+.
- From the external auth provider: Issuer URL, Client ID, Client Secret,
  and (for Keycloak) Realm name.
- Cluster administrator access (needed to create secrets and configure
  `GatewayConfig`).

## Steps

1. Verify the OpenShift authentication type is OIDC:
   ```
   oc get authentication.config/cluster -o jsonpath='{.spec.type}'
   ```
   Expect `OIDC`.
2. Verify the OIDC provider name:
   ```
   oc get authentication.config/cluster -o jsonpath='{.spec.oidcProviders[*].name}'
   ```
3. Verify `kube-apiserver` has finished rolling out the change:
   ```
   oc get co kube-apiserver
   ```
   Expect `AVAILABLE=True`, `PROGRESSING=False`, `DEGRADED=False`. The
   rollout can take 20+ minutes — wait for all nodes before continuing.
4. Set environment variables from your IdP's details:
   ```
   IDP_DOMAIN="<keycloak.example.com>"
   IDP_REALM="<your-realm>"
   IDP_CLIENT_ID="<your-client-id>"
   IDP_CLIENT_SECRET="<your-client-secret>"
   IDP_ISSUER_URL="<https://keycloak.example.com/realms/your-realm>"
   ```
5. Create the client secret in `openshift-ingress`:
   ```
   oc create secret generic idp-client-secret \
     --from-literal=clientSecret=$IDP_CLIENT_SECRET \
     -n openshift-ingress
   ```
6. Patch `GatewayConfig` with the OIDC details:
   ```
   oc patch gatewayconfig default-gateway --type='merge' -p='{
     "spec": {
       "oidc": {
         "issuerURL": "'$IDP_ISSUER_URL'",
         "clientID": "'$IDP_CLIENT_ID'",
         "clientSecretRef": {
           "name": "idp-client-secret",
           "key": "clientSecret"
         }
       }
     }
   }'
   ```
7. Confirm the secret and `GatewayConfig`:
   ```
   oc get secret keycloak-client-secret -n openshift-ingress
   oc get gatewayconfig default-gateway -o jsonpath='{.spec.oidc}'
   ```

## Verification

1. `oc get consolelink` → open the Console link → log in with OIDC
   credentials → confirm redirect to the OIDC login page, successful
   redirect back to the Gateway, and that RHOAI components (Dashboard,
   Notebooks) are reachable.
2. `oc get gatewayconfig default-gateway -o yaml` → `spec.oidc` present,
   and `Ready`/`ProvisioningSucceeded` conditions both `status: "True"`.
3. `oc get deployment kube-auth-proxy -n openshift-ingress` → `1/1` ready.
4. `oc get gateway data-science-gateway -n openshift-ingress` →
   `PROGRAMMED: True` with a valid address.
5. `curl -s https://keycloak.example.com/realms/your-realm/.well-known/openid-configuration`
   → returns the OIDC discovery JSON (`issuer`, `authorization_endpoint`,
   `token_endpoint`, etc.).

## Next steps — authorizing IdP groups

External OIDC authentication alone does not grant project access; the
cluster administrator must map IdP groups to OpenShift `ClusterRole`s:

1. Create a read/list-projects `ClusterRole`:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: ClusterRole
   metadata:
     name: odh-projects-read
   rules:
   - apiGroups: ["project.openshift.io"]
     resources: ["projects"]
     verbs: ["get","list"]
   ```
2. Bind it to the IdP group (e.g. `odh-users`):
   ```
   oc adm policy add-cluster-role-to-group odh-projects-read odh-users
   ```
3. Grant project self-provisioning:
   ```
   oc adm policy add-cluster-role-to-group self-provisioner odh-users
   ```

## Security considerations

- Store OIDC client secrets securely and rotate them regularly.
- Consider network policies restricting access to the authentication
  proxy.
- All OIDC communication should use TLS.
- `kube-auth-proxy` validates tokens, but confirm your OIDC provider uses
  appropriate token lifetimes.
- Enable audit logging for authentication events.

## See also

[[configuring-service-account-token-authentication]] for the programmatic
path, [[configuring-custom-ca-for-oidc]] for a self-signed IdP
certificate, [[troubleshooting-centralized-authentication-service]] for
failure symptoms.
