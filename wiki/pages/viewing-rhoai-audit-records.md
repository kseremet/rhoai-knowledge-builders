---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L1280-1311
  - raw/rhoai/3.5/managing-openshift-ai.md#L2246-2277
verified: 2026-08-25
confidence: documented
---

# Viewing RHOAI audit records

Uses OpenShift's own audit logging to review the history of user changes
to the [[datasciencecluster]] (DSC) and [[dscinitialization]] (DSCI) custom
resources — distinct from [[configuring-the-rhoai-operator-logger]], which
controls the Operator's own runtime log verbosity, not the audit trail.

This procedure is documented near-identically in both the install guide
and the "Managing OpenShift AI" guide's Chapter 13 ("Viewing logs and
audit records") — same `oc adm node-logs`/`jq` commands, same filter
logic. No content differences found between the two sources; this page
merges both citations rather than duplicating the page.

Audit logging is enabled by default in standard OpenShift cluster
configs. **Exception**: on Red Hat OpenShift Service on AWS, audit logging
is disabled by default (the Elasticsearch log store doesn't provide secure
audit-log storage) — see "Logging" in the ROSA docs (not ingested) to
configure forwarding.

## Preconditions

- Cluster administrator access.
- OpenShift CLI (`oc`) installed.

## Steps

1. Log in as a cluster administrator (if not already):
   `oc login <openshift_cluster_url> -u <admin_username> -p <password>`
2. Set the OpenShift audit log policy to `WriteRequestBodies` or more
   comprehensive, to capture full changed-resource content (see
   "Configuring the audit log policy" in OCP docs, not ingested).
3. Fetch audit logs from control-plane nodes:
   ```
   oc adm node-logs --role=master --path=kube-apiserver/ \
     | awk '{ print $1 }' | sort -u \
     | while read node ; do
         oc adm node-logs $node --path=kube-apiserver/audit.log < /dev/null
       done \
     | grep opendatahub > /tmp/kube-apiserver-audit-opendatahub.log
   ```
4. Filter for DSC/DSCI change events (excluding the Operator's own
   reconciliation reads):
   ```
   jq 'select((.objectRef.apiGroup == "dscinitialization.opendatahub.io" or
              .objectRef.apiGroup == "datasciencecluster.opendatahub.io") and
             .user.username != "system:serviceaccount:redhat-ods-operator:redhat-ods-operator-controller-manager" and
             .verb != "get" and .verb != "watch" and .verb != "list")' \
     < /tmp/kube-apiserver-audit-opendatahub.log
   ```

## Verification

The commands return relevant log entries.

## Tip

To configure log retention, see "Logging" in the OpenShift docs (not
ingested).
