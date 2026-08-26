---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1966-2026
verified: 2026-08-25
confidence: documented
---

# Enabling the RHOAI observability stack

**Technology Preview, as of 3.5.** Turns on RHOAI's centralized platform
observability: an integrated stack with the OpenTelemetry Collector (OTC)
for data ingestion, Prometheus for metrics, and Red Hat build of Tempo for
distributed tracing. Enables common health metrics/alerts for RHOAI
components, and integration paths to external observability tools — see
[[exporting-metrics-to-external-tools]] and
[[viewing-traces-in-external-tools]].

Enabled on the [[dscinitialization]] (DSCI) CR — a cluster administrator
must do this explicitly.

## Preconditions

- Cluster administrator privileges.
- RHOAI installed.
- These Operators installed: **Cluster Observability Operator** (deploys
  Prometheus + Alertmanager), **Tempo Operator** (Tempo backend for
  tracing), **Red Hat build of OpenTelemetry** (deploys the OTC).

## Steps

1. Log in to the OpenShift web console as a cluster administrator.
2. Installed Operators page (**Ecosystem** → **Installed Operators** on
   OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19) →
   **Red Hat OpenShift AI** → **DSCInitialization** tab → the default
   instance (e.g. `default-dsci`) → **YAML**.
3. Under `spec.monitoring`, set `managementState: Managed` and configure
   metrics/alerting/tracing:
   ```yaml
   spec:
     monitoring:
       managementState: Managed        # Required: enables and manages the stack
       namespace: redhat-ods-monitoring # Required: namespace for monitoring components
       alerting: {}                     # Alertmanager config; {} = defaults
       metrics:                         # Prometheus config
         replicas: 1                    # Optional: number of Prometheus instances
         resources:                     # CPU/memory for Prometheus pods
           cpulimit: 500m
           cpurequest: 100m
           memorylimit: 512Mi
           memoryrequest: 256Mi
         storage:
           size: 5Gi                    # Required: Prometheus storage size
           retention: 90d                # Required: metrics retention period
         exporters: {}                   # External metrics exporters
       traces:                          # Tempo config
         sampleRatio: '0.1'              # Optional: fraction of traces sampled
         storage:
           backend: pv                   # Required: pv, s3, or gcs
           retention: 2160h              # Optional: trace retention (hours)
         exporters: {}                   # External traces exporters
   ```
4. **Save**.

## Verification

**Workloads** → **Pods** → project `redhat-ods-monitoring` — confirm
running pods including:
```
alertmanager-data-science-monitoringstack-#   2/2  Running
data-science-collector-collector-#            1/1  Running
prometheus-data-science-monitoringstack-#     2/2  Running
tempo-data-science-tempomonolithic-#          1/1  Running
thanos-querier-data-science-thanos-querier-#  2/2  Running
```

## Next steps

- [[collecting-metrics-from-user-workloads]] — opt individual workloads
  into scraping.
- [[enabling-observability-dashboard]] — surface this in the RHOAI
  dashboard UI.
