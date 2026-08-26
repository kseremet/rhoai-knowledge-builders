---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L890-921
verified: 2026-08-25
confidence: documented
---

# Using the cluster-wide CA bundle for the model serving platform

By default, RHOAI's model serving platform (KServe, via [[datasciencecluster]]'s
`kserve` component) generates its own self-signed certificate at install
time for model-server endpoints. Cluster-wide certificates, once
configured, are already used by default for other endpoint types (e.g.
route endpoints). This procedure makes model-serving endpoints use the
same certificate already configured for the OpenShift cluster, instead of
the auto-generated self-signed one.

## Preconditions

- Cluster administrator access.
- Cluster-wide certificates already configured on OpenShift.

## Steps

1. Log in to the web console as a cluster administrator.
2. Open the `openshift-ingress` project → **YAML**.
3. Search for a secret with "cert" in its name (e.g.
   `rhods-internal-primary-cert-bundle-secret`) — it holds `tls.cert` and
   `tls.key`, used for all OpenShift Routes.
4. Copy the secret reference.
5. Open the `istio-system` project, create a new YAML file, and paste in
   the reference, trimmed to the essentials:
   ```yaml
   kind: Secret
   apiVersion: v1
   metadata:
     name: rhods-internal-primary-cert-bundle-secret
   data:
     tls.crt: >-
       LS0tLS1CRUd...
     tls.key: >-
       LS0tLS1CRUd...
   type: kubernetes.io/tls
   ```
   (replace the secret name with your own).
6. Save the file in the `istio-system` project.
7. Installed Operators → **Data Science Cluster** → `default-dsc` →
   **YAML**.
8. Edit the `kserve` section to reference the secret:
   ```yaml
   kserve:
     devFlags: {}
     managementState: Managed
     serving:
       ingressGateway:
         certificate:
           secretName: rhods-internal-primary-cert-bundle-secret
           type: Provided
       managementState: Managed
       name: knative-serving
   ```

## Gap

No explicit verification step is given in this guide for this procedure.
