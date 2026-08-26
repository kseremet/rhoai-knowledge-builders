---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L990-1129
verified: 2026-08-25
confidence: documented
---

# OGX server CA bundle configuration reference

Reference detail for [[ogx-server]]'s `spec.server.tlsConfig.caBundle`
field: alternative configuration patterns beyond the recommended one
covered in [[configuring-ca-bundle-for-ogx-server]], validation rules,
enforced limits, failure conditions, and how client code inside the pod
uses the resulting bundle.

## `caBundle` subfields

- `configMapName` (required) — source config map name.
- `configMapNamespace` (optional) — defaults to the `OGXServer` CR's
  namespace. Cross-namespace reference requires the OGX Operator service
  account to have read access to the source namespace.
- `configMapKeys` (optional) — keys in the source config map holding CA
  data, concatenated together. Defaults to `ca-bundle.crt` alone.

## Alternative reference patterns

**Dedicated config map** (certificates specific to this OGX server, in the
same namespace as the CR):
```yaml
apiVersion: ogx.io/v1beta1
kind: OGXServer
metadata:
  name: my-ogx
  namespace: my-ogx-namespace
spec:
  distribution:
    name: rh-dev
  workload:
    replicas: 1
  tlsConfig:
    caBundle:
      configMapName: external-llm-ca
```
Create it with:
```
oc create configmap <ca-bundle-configmap> \
  --from-file=ca-bundle.crt=<path/to/ca-bundle.crt> \
  -n <namespace>
```
(concatenate multiple CA certs' PEM blocks into the file first, to include
more than one).

**Config map in a different namespace**:
```yaml
spec:
  server:
    tlsConfig:
      caBundle:
        configMapName: enterprise-ca-bundle
        configMapNamespace: security-system
```

## Validation rules

Only PEM blocks of type `CERTIFICATE` are processed; other types (e.g.
`PRIVATE KEY`) are silently ignored. Each block must parse as a valid X.509
certificate. The combined bundle must contain at least one valid
`CERTIFICATE` block, or the Operator refuses to create the managed config
map and reports a validation failure on the CR status.

## Limits

| Limit | Value | Notes |
|---|---|---|
| Maximum bundle size | 10 MB | total size of the concatenated PEM bundle built from the selected keys; larger bundles are rejected |
| Maximum certificate count | 1000 | valid X.509 `CERTIFICATE` blocks; more are rejected |
| Accepted PEM block type | `CERTIFICATE` | other types ignored without error |

The standard Kubernetes 1 MB config map size limit still applies to the
*source* config map — the 10 MB limit is on the Operator's concatenated
output.

## CA bundle validation failure conditions

Any of the following causes `DeploymentReady` to go `"False"` with a
descriptive `message`, and the Operator does not deploy the pod with an
invalid bundle:

- Referenced config map does not exist in the specified namespace.
- Selected key does not exist in the source config map.
- Selected keys contain no valid X.509 `CERTIFICATE` PEM blocks.
- Concatenated bundle exceeds 10 MB.
- Concatenated bundle exceeds 1000 certificates.
- OGX Operator service account lacks read access to the source config map.

Correcting the source config map or the CR reference triggers automatic
reconciliation and a managed-config-map update — no manual retry needed.

## Using the bundle from client code

Inside the pod, the bundle lives at
`/etc/ssl/certs/ca-bundle/ca-bundle.crt`, and `SSL_CERT_FILE` on the server
container points to it — most Python HTTP libraries (`httpx`, `requests`)
honor this automatically. Libraries that don't must be passed the path
explicitly, e.g.:
```python
from ogx_client import OgxClient
import httpx

http_client = httpx.Client(verify="/etc/ssl/certs/ca-bundle/ca-bundle.crt")
client = OgxClient(
    base_url="https://my-ogx.my-ogx-namespace.svc:8321",
    http_client=http_client,
)
```
