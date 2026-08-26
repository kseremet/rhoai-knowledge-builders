---
type: error
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1454-1473
verified: 2026-08-25
confidence: documented
---

# The RHOAI dashboard is not accessible

## Symptom

After installing RHOAI, the `redhat-ods-applications`,
`redhat-ods-monitoring`, and `redhat-ods-operator` project namespaces all
show `Active`, but the dashboard itself cannot be reached — the underlying
cause is an error in one of the pods backing it.

This is distinct from a user simply not knowing the dashboard URL — see
[[accessing-the-rhoai-dashboard]] for that — and from Operator-install-time
failures, which are [[troubleshooting-rhoai-operator-installation|covered
separately]].

## Which components can produce this

Any pod in `redhat-ods-applications` (the dashboard and other required
[[red-hat-openshift-ai]] components), `redhat-ods-monitoring`, or
`redhat-ods-operator` (the [[rhoai-operator]] itself) — this guide does not
narrow it further than "a pod is erroring", so the diagnostic flow below is
the whole documented process: find the errored pod first, then read what
it says.

## Diagnostic commands / steps, in order

1. Web console → **Administrator** perspective.
2. **Workloads** → **Pods**.
3. **Project**: **All Projects**.
4. Click **Filter** → select every status checkbox **except** **Running**
   and **Completed**. The page now shows only pods in an error state.

## Causes and fixes

The guide does not enumerate specific root causes for this symptom (unlike
the Chapter 10 operator-install errors, which have named causes) — it
documents only the generic pod-triage procedure:

- If the pod's **Status** column shows a link, click it for more detail and
  targeted troubleshooting steps for that specific failure.
- If **Status** has no link, open the pod's details page directly and check
  the **Logs** tab.

## Telling causes apart

Not covered by this guide beyond "read whichever pod is erroring" — the
actual root cause (image pull failure, crash loop, resource pressure,
missing config, etc.) is whatever that pod's own logs/events show. This is
a real gap: the guide gives the *triage* procedure but no differentiated
symptom → cause → fix mapping for dashboard-specific failures the way
Chapter 10 does for operator-install failures. If a specific error string
is found in a pod's logs during this triage, check
[[troubleshooting-rhoai-operator-installation]] first (several of its
symptoms are also read from pods in these same namespaces) before escalating
to Red Hat Support.
