---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L2135-2163
verified: 2026-08-25
confidence: documented
---

# Viewing traces in external tracing platforms

When tracing is enabled on the [[dscinitialization]] (DSCI) CR as part of
[[enabling-observability-stack]], RHOAI deploys Red Hat build of Tempo as
the tracing backend and the OpenTelemetry Collector (OTC) to receive/route
trace data. This connects an external visualization tool (Grafana,
Jaeger) to that data.

## Preconditions

- A cluster administrator has enabled tracing in the DSCI — see
  [[enabling-observability-stack]].
- Access to the monitoring namespace (e.g. `redhat-ods-monitoring`).
- Network access or cluster administrator privileges to create a route or
  port-forward.
- The target application is instrumented with an OpenTelemetry SDK/library.

## Steps

### Find the OpenTelemetry Collector endpoint

Applications send traces to the OTC via OTLP.

1. **Networking** → **Services** → project `redhat-ods-monitoring`.
2. Locate the `data-science-collector` (or similarly-named) Service.
3. Use its Service name or ClusterIP as the OTLP endpoint. Export to port
   `4317` (gRPC) or `4318` (HTTP), e.g.:
   ```
   OTEL_EXPORTER_OTLP_ENDPOINT=http://data-science-collector.redhat-ods-monitoring.svc.cluster.local:4318
   ```

### Connect a visualization tool to the Tempo query service

1. **Networking** → **Services** → project `redhat-ods-monitoring`.
2. Locate `tempo-query` or `tempo-query-frontend`.
3. Expose it externally — either create a route, or port-forward:
   ```
   oc port-forward svc/tempo-query-frontend 3200:3200 -n redhat-ods-monitoring
   ```
4. Point the visualization tool at the Tempo Query API, e.g.
   `http://localhost:3200`.

## Verification

1. Confirm the instrumented application is generating/exporting traces.
2. `oc get pods -n redhat-ods-monitoring | grep collector` — the
   `data-science-collector-collector-*` pod is `Running`.
3. Confirm new traces appear in the visualization tool.
