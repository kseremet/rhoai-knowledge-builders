---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L2063-2099
verified: 2026-08-25
confidence: documented
---

# Collecting metrics from user workloads

Once [[enabling-observability-stack|the observability stack is enabled]],
metric collection is *available* but not automatic for every workload —
Prometheus only scrapes pods carrying a specific label. Opts a workload
(workbench, training job, inference service) into the stack.

Apply the `monitoring.opendatahub.io/scrape=true` label **only** to
workloads that actually expose metrics and that you want monitored — don't
apply it to Operator-managed workloads, since the Operator may strip it
during reconciliation.

## Preconditions

- A cluster administrator has enabled the observability stack — see
  [[enabling-observability-stack]].
- RHOAI administrator privileges, or project ownership.
- The target workload exposes a `/metrics` endpoint (e.g. a workbench
  server or model-service pod).
- Access to the project the workload runs in.

## Steps

1. Log in to the OpenShift web console as a cluster administrator or
   project owner.
2. **Workloads** → **Deployments** → select the project.
3. Click the target deployment's name → **YAML** tab.
4. Add the label under `spec.template.metadata.labels`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: <example_name>
     namespace: <example_namespace>
   spec:
     template:
       metadata:
         labels:
           monitoring.opendatahub.io/scrape: 'true'
       # ...
   ```
5. **Save**. OpenShift rolls out a new ReplicaSet/pods with the label;
   scraping begins once the new pods start.

## Verification

1. Access Prometheus: **Networking** → **Routes** → project
   `redhat-ods-monitoring` → open the route associated with the Prometheus
   service (e.g. `data-science-prometheus-route`). Or, port-forward:
   ```
   oc get pods -n redhat-ods-monitoring -l prometheus=data-science-monitoringstack
   oc port-forward <prometheus-pod-name> 9090:9090 -n redhat-ods-monitoring
   ```
   then open `http://localhost:9090`.
2. Search for a metric exposed by the workload — it should appear if the
   label was applied correctly.
