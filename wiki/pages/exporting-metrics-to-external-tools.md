---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L2100-2134
verified: 2026-08-25
confidence: documented
---

# Exporting metrics to external observability tools

Sends RHOAI operational metrics to an external platform (Grafana,
Prometheus, or any OpenTelemetry-compatible backend), alongside metrics
from other systems. Configured via `.spec.monitoring.metrics.exporters` on
the [[dscinitialization]] (DSCI) CR — populating it makes the
OpenTelemetry Collector (OTC), deployed as part of
[[enabling-observability-stack]], add each exporter to its metrics
pipeline. Leaving it empty/undefined means metrics only reach the
in-cluster Prometheus instance.

## Preconditions

- Cluster administrator privileges.
- The observability stack enabled — see [[enabling-observability-stack]].
- The external platform can receive metrics over a supported export
  protocol, and you know its receiver endpoint URL.

## Steps

1. Log in to the OpenShift web console as a cluster administrator.
2. Installed Operators page (**Ecosystem** → **Installed Operators** on
   OpenShift 4.20+; **Operators** → **Installed Operators** on 4.19) →
   **Red Hat OpenShift AI** → **DSCInitialization** tab → the default DSCI
   instance (e.g. `default-dsci`) → **YAML**.
3. Under `spec.monitoring.metrics`, add an `exporters` list:
   ```yaml
   spec:
     monitoring:
       metrics:
         exporters:
         - name: <external_exporter_name>
           type: <type>
           endpoint: https://example-otlp-receiver.example.com:4317
   ```
   - `name`: unique descriptive name — avoid reserved names like
     `prometheus` or `otlp/tempo`.
   - `type`: `otlp` (OpenTelemetry-compatible backends, gRPC or HTTP) or
     `prometheusremotewrite` (Prometheus remote-write protocol).
   - `endpoint`: full receiver URL. OTLP typically uses port `4317`
     (gRPC) or `4318` (HTTP); Prometheus remote write typically ends
     `/api/v1/write`.
4. **Save**. The OTC reloads and starts forwarding to the new endpoint.

## Verification

1. `oc get pods -n redhat-ods-monitoring` — `data-science-collector-collector-*`
   pods restart and reach `Running`.
2. Confirm new RHOAI metrics appear in the external platform.

**Note**: removing `.spec.monitoring.metrics.exporters` from the DSCI
reverts the OTC to collecting only for the in-cluster Prometheus instance.
