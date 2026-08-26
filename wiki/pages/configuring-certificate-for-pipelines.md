---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L831-863
verified: 2026-08-25
confidence: documented
---

# Configuring a certificate for AI pipelines

Provides a CA bundle used *only* by AI pipelines, via the
`DataSciencePipelinesApplication` (DSPA) CR's `spec.apiServer.cABundle`
field — for when the pipeline server needs to reach an external system
using a self-signed/custom certificate beyond what the cluster-wide bundle
covers.

This is one of three options for getting a certificate to AI pipelines; the
other two are [[adding-certificates-to-cluster-wide-ca-bundle]] and
[[adding-certificates-to-custom-ca-bundle]] (both apply more broadly, not
just to pipelines).

## Preconditions

- Cluster administrator access.
- A self-signed certificate saved to a file (e.g. `example-ca.crt`).
- A pipeline server already configured and available (see "Configuring a
  pipeline server", not yet ingested).
- OpenShift CLI (`oc`) installed.

## Steps

1. Log in to the web console as a cluster administrator.
2. Create a config map with the bundle, in the target pipeline's project:
   Workloads → ConfigMaps → **Create ConfigMap** → YAML view:
   ```yaml
   kind: ConfigMap
   apiVersion: v1
   metadata:
     name: custom-ca-bundle
   data:
     ca-bundle.crt: |
       # contents of ca-bundle.crt
   ```
3. Update the DSPA's `spec.apiServer.cABundle`:
   Home → Search → `DataSciencePipelinesApplication` → select it → YAML:
   ```yaml
   apiVersion: datasciencepipelinesapplications.opendatahub.io/v1
   kind: DataSciencePipelinesApplication
   metadata:
     name: data-science-dspa
   spec:
     apiServer:
       cABundle:
         configMapName: custom-ca-bundle
         configMapKey: ca-bundle.crt
   ```
   Optional overrides (default mount path `/dsp-custom-certs`, default file
   `dsp-ca.crt`):
   ```yaml
   spec:
     apiServer:
       caBundleFileMountPath: /tmp/custom-dsp-certs
       caBundleFileName: custom-dsp-ca.crt
   ```
4. **Save** — the pipeline server pod redeploys automatically.

**Note**: the pipeline controller always creates a managed config map named
`dsp-trusted-ca-<dspa-name>` (e.g. `dsp-trusted-ca-dspa` for a DSPA named
`dspa`) — this is what the running pod actually mounts, not the source
config map directly. This managed config map is created even without
setting `cABundle`: with `podToPodTLS` enabled (the default), the cluster
service CA is always added to it; setting `cABundle` adds your source
config map's certs to the same managed config map.

## Verification

- `oc get configmap dsp-trusted-ca-<dspa-name> -n <namespace>` — the
  managed config map exists.
- `oc get pods -n <namespace> | grep ds-pipeline-<dspa-name>-` — find the
  pipeline API server pod.
- `oc exec -n <namespace> <pod> -c ds-pipeline-api-server -- ls -l /dsp-custom-certs/`
  — confirms the bundle is mounted at the default (or configured) path.
- `oc exec -n <namespace> <pod> -c ds-pipeline-api-server -- cat /dsp-custom-certs/dsp-ca.crt | head -5`
  — output starts with `-----BEGIN CERTIFICATE-----`.
- `oc exec -n <namespace> <pod> -c ds-pipeline-api-server -- env | grep -iE CABUNDLE|SSL_CERT_DIR|ARTIFACT_COPY`
  — `SSL_CERT_DIR` lists `/dsp-custom-certs` first (e.g.
  `SSL_CERT_DIR=/dsp-custom-certs:/etc/ssl/certs:/etc/pki/tls/certs`), plus
  `ARTIFACT_COPY_STEP_CABUNDLE_*` variables.
