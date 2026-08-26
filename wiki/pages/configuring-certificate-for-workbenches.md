---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L864-889
verified: 2026-08-25
confidence: documented
---

# Configuring a certificate for workbenches

[[rhoai-workbench]]s automatically trust the cluster-wide and custom CA
bundles once configured (see [[how-rhoai-handles-certificates]]). This
procedure covers applying an already-configured bundle to a workbench, and
pointing a package that ignores the preset environment variables at the
bundle explicitly.

## Preconditions

- Logged in to RHOAI.
- Cluster-wide certificates already configured (see
  [[adding-certificates-to-cluster-wide-ca-bundle]]).
- Access to a workbench.

## What gets mounted

The combined bundle is mounted at
`/etc/pki/tls/custom-certs/ca-bundle.crt`, and these environment variables
are preset to point at it:

| Variable | Consumed by |
|---|---|
| `SSL_CERT_FILE` | OpenSSL and many Python libraries |
| `REQUESTS_CA_BUNDLE` | Python `requests` |
| `PIP_CERT` | `pip` |
| `GIT_SSL_CAINFO` | `git` over HTTPS |
| `KF_PIPELINES_SSL_SA_CERTS` | the Elyra extension → Kubeflow Pipelines SDK (`kfp`), when submitting pipelines from the visual editor |

## Steps

1. Apply the certificate to the workbench:
   - **New workbench**: create it (see [[creating-a-workbench]]) *after*
     cluster-wide certificates are configured — it trusts the bundle
     automatically.
   - **Existing workbench**: stop and restart it (see "Starting a
     workbench", not yet ingested). By default self-signed certs only
     apply to workbenches created *after* configuring cluster-wide certs —
     restarting applies the current bundle.
2. Optional — for a package that does **not** read any preset env var
   (e.g. calling `kfp` directly rather than through Elyra), pass the
   certificate path explicitly:
   ```python
   from kfp.client import Client
   with open(sa_token_file_path, 'r') as token_file:
       bearer_token = token_file.read()
   client = Client(
       host='https://<pipeline_server_route>/',
       existing_token=bearer_token,
       ssl_ca_cert='/etc/pki/tls/custom-certs/ca-bundle.crt'
   )
   print(client.list_experiments())
   ```
   (Elyra itself reads `KF_PIPELINES_SSL_SA_CERTS` automatically when
   submitting from the visual editor — this explicit-path step is only for
   direct `kfp` client use.)

## Verification

- In a workbench terminal:
  `env | grep -E 'SSL_CERT_FILE|REQUESTS_CA_BUNDLE|PIP_CERT|GIT_SSL_CAINFO|KF_PIPELINES_SSL_SA_CERTS'`
  — each points to `/etc/pki/tls/custom-certs/ca-bundle.crt`.
- `head -5 /etc/pki/tls/custom-certs/ca-bundle.crt` — output starts with
  `-----BEGIN CERTIFICATE-----`.
