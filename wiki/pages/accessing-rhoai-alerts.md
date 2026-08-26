---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L2164-2190
verified: 2026-08-25
confidence: documented
---

# Accessing built-in RHOAI alerts

The observability stack (see [[enabling-observability-stack]]) deploys a
Prometheus Alertmanager instance with a common set of built-in alerts for
RHOAI components — operator downtime, crashlooping pods, unresponsive
services. Alertmanager is internal by default (no route); this accesses it
locally via `oc` port-forwarding.

## Preconditions

- RHOAI administrator privileges.
- The observability stack enabled — see [[enabling-observability-stack]].
- Know the monitoring namespace (e.g. `redhat-ods-monitoring`).
- OpenShift CLI (`oc`) installed.

## Steps

1. `oc login https://api.198.51.100.10:6443` (as a cluster administrator).
2. Confirm Alertmanager pods are running:
   ```
   oc get pods -n redhat-ods-monitoring | grep alertmanager
   ```
   Example:
   ```
   alertmanager-data-science-monitoringstack-0 2/2 Running 0 2h
   alertmanager-data-science-monitoringstack-1 2/2 Running 0 2h
   ```
3. Confirm the ClusterIP service on port 9093:
   ```
   oc get svc -n redhat-ods-monitoring | grep alertmanager
   ```
4. Port-forward:
   ```
   oc port-forward svc/data-science-monitoringstack-alertmanager 9093:9093 -n redhat-ods-monitoring
   ```
5. Open `http://localhost:9093` in a browser.

## Verification

The Alertmanager web interface opens at `http://localhost:9093` and shows
active alerts for RHOAI components.
