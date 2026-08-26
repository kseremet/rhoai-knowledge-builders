---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1733-1766
verified: 2026-08-25
confidence: documented
---

# Configuring service account token authentication for the Gateway

The programmatic/CLI counterpart to
[[configuring-oidc-centralized-authentication|OIDC user authentication]] on
the [[centralized-authentication-service]] — for automation and CLI
tools, not interactive browser access.

## Preconditions

- Cluster administrator access.
- RHOAI installed with Gateway API support on OpenShift Container Platform
  4.19.9+.

## Steps

1. Create a `ServiceAccount` for the app/CLI tool:
   ```
   oc create serviceaccount my-app-sa -n <your-namespace>
   ```
   (e.g. `<your-namespace>` = `redhat-ods-applications`.)
2. Grant it RBAC access to the target service, e.g. via `kube-rbac-proxy`:
   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: my-app-service-access
   rules:
   - apiGroups: [""]
     resources: ["services/proxy"]
     resourceNames: ["my-service"]
     verbs: ["get", "create"]
   ```
   ```
   oc create -f role.yaml -n <your-namespace>
   oc create rolebinding my-app-service-access \
     --role=my-app-service-access \
     --serviceaccount=<your-namespace>:my-app-sa \
     -n <your-namespace>
   ```
3. Generate a token:
   ```
   TOKEN=$(oc create token my-app-sa -n <your-namespace> --duration=1h)
   ```
   Kubernetes enforces a minimum duration of 10 minutes.
4. Get the Gateway URL:
   ```
   GATEWAY_URL=$(oc get route data-science-gateway -n openshift-ingress -o jsonpath='{.spec.host}')
   ```
5. Access the service through the Gateway with the bearer token:
   ```
   curl -k -H "Authorization: Bearer ${TOKEN}" \
     "https://${GATEWAY_URL}/your-service-path"
   ```

## Verification

```
curl -k -i -H "Authorization: Bearer ${TOKEN}" \
  "https://${GATEWAY_URL}/your-service-path"
```
Successful authentication returns HTTP 200. Invalid/missing tokens return
HTTP 403 or a redirect to authentication.

## Security considerations

- Give tokens limited lifetimes via `--duration`, sized to your security
  needs.
- Grant service accounts only the permissions they need — avoid
  cluster-admin or overly broad RBAC.
- Never commit tokens to source control; use secrets management.
- Rotate tokens for long-running services, regenerating before expiry.

## See also

[[troubleshooting-centralized-authentication-service]] — token-duration
and 401-Unauthorized symptoms tied to this procedure.
