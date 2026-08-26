---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L226-256
verified: 2026-08-25
confidence: documented
---

# Configuring custom namespaces for RHOAI

Replaces one or more of [[rhoai-operator]]'s three predefined namespaces
(`redhat-ods-operator`, `redhat-ods-applications`, `rhods-notebooks`) with
custom names, for organizations with naming policies/conventions. Must be
done **before** installing the Operator — a custom workbench namespace
cannot be changed after the Operator is installed.

## Preconditions

- Cluster administrator access.
- OpenShift CLI (`oc`) installed.
- The Red Hat OpenShift AI Operator is **not yet** installed.

## Steps

All three are optional and independent — do only the ones you need.

1. Log in as a cluster administrator: `oc login <cluster_url> -u <admin_username> -p <password>`
2. **Custom operator namespace** (optional):
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: <operator-namespace>
   ```
   `$ oc create -f operator-namespace.yaml` — use this namespace name in
   place of `redhat-ods-operator` in [[installing-the-rhoai-operator]].
3. **Custom applications namespace** (optional):
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: <applications-namespace>
     labels:
       opendatahub.io/application-namespace: 'true'
   ```
   `$ oc create -f applications-namespace.yaml` — the
   `opendatahub.io/application-namespace: 'true'` label is required.
4. **Custom workbench namespace** (optional):
   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: <workbench-namespace>
   ```
   `$ oc create -f workbench-namespace.yaml` — use this value for
   `spec.workbenches.workbenchNamespace` on the [[datasciencecluster]] CR
   during [[installing-rhoai-components]]. **Cannot be changed** after the
   Operator is installed.

## Verification

Each `oc create -f ...` prints `namespace/<name> created`.

## Next step

[[installing-the-rhoai-operator]]
