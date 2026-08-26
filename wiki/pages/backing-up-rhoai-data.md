---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1938-1965
verified: 2026-08-25
confidence: documented
---

# Backing up RHOAI data

Two independent backup concerns: the storage data on persistent volume
claims (PVCs), and the OpenShift cluster itself.

## Backing up storage data

Best practice to do regularly. Particularly important before deleting a
user (see [[deleting-rhoai-users]]) and before uninstalling RHOAI (see
[[uninstalling-rhoai]]) — **all PVCs are deleted when RHOAI is
uninstalled**.

For platform-specific PVC backup, see "OADP Application backup and
restore" and "Understanding persistent storage" in the OpenShift
Container Platform documentation (not ingested).

## Backing up your cluster

Before upgrading or uninstalling RHOAI, back up cluster data so it can be
restored later if needed. See "Backup and restore" in the OpenShift
Container Platform documentation (not ingested).

## See also

[[deleting-rhoai-users]] — the user-teardown flow this backup step feeds
into; [[uninstalling-rhoai]] — the other operation this guards against
data loss for.
