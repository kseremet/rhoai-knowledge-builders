---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1231-1279
  - raw/rhoai/3.5/managing-openshift-ai.md#L2197-2245
verified: 2026-08-25
confidence: documented
---

# Configuring the RHOAI Operator logger

Changes the log level of the `rhods-operator` pod itself via
`spec.devFlags.logLevel` on the [[dscinitialization]] DSCI object, and
covers streaming the resulting logs. This log level applies **only** to
the Operator pod, not to the individual RHOAI components it manages.

This procedure is documented near-identically in both the install guide
(as part of the DSCI reference) and the "Managing OpenShift AI" guide's
Chapter 13 ("Viewing logs and audit records") — same fields, same
commands, same log-level table. No content differences found between the
two sources; this page merges both citations rather than duplicating the
page.

## Log levels

| Level | Severity | Verbosity | Notes |
|---|---|---|---|
| `error` | N/A | Low | error messages only |
| `info` | `0` | Medium | standard info + error; **default** when unset |
| `debug` | `-1` | High | debug + info + error |
| custom integer | `1`+ | Very High | fine-grained controller execution tracing; higher = more detail |

## Preconditions

- Administrator access to `DSCInitialization` resources.
- OpenShift CLI (`oc`) installed.

## Steps — web console

1. Log in as a cluster administrator.
2. Installed Operators → **DSC Initialization** tab → `default-dsci` →
   **YAML**.
3. Set:
   ```yaml
   apiVersion: dscinitialization.opendatahub.io/v2
   kind: DSCInitialization
   metadata:
     name: default-dsci
   spec:
     devFlags:
       logLevel: debug
   ```
4. **Save**.

## Steps — CLI (alternative)

```
oc patch dsci default-dsci -p {"spec":{"devFlags":{"logLevel":"debug"}}} --type=merge
```

## Verification

- `error`: logs generate infrequently, only critical failures/errors.
- `info` (or unset): standard-frequency logs with info + error messages.
- `debug`: frequent logs with debug + info + error messages.
- Numeric (`1`, `2`, `3`, ...): very high-frequency, deep controller
  execution tracing — progressively more detail at higher integers.

## Viewing the Operator logs

```
for pod in $(oc get pods -l name=rhods-operator -n redhat-ods-operator -o name); do
  oc logs -f "$pod" -n redhat-ods-operator &
done
```
Press Ctrl+C to stop viewing; `kill $(jobs -p)` to fully stop all log
streams. Equivalent in the web console: Workloads → Pods →
`redhat-ods-operator` project → pod → **Logs** tab.

## See also

[[troubleshooting-rhoai-operator-installation]] and
[[rhoai-dashboard-not-accessible]] both rely on reading these same
`rhods-operator` (or component) pod logs to diagnose install failures.
[[viewing-rhoai-audit-records]] covers the separate audit-trail mechanism
(who changed the DSC/DSCI CRs), not this Operator log-level setting.
