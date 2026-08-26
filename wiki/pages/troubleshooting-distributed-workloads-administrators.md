---
type: error
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1537-1642
verified: 2026-08-25
confidence: documented
---

# Troubleshooting distributed workloads (administrators)

Symptoms on Ray/PyTorch distributed-training workloads, from the
administrator side — for the underlying quota/queue mechanics see
[[configuring-quota-management-for-distributed-workloads]] and
[[kueue-workload-management]]. See also [[troubleshooting-kueue]] and
"Troubleshooting common problems with distributed workloads for users"
(not ingested). If undocumented here or in the release notes, contact Red
Hat Support.

## A user's Ray cluster is in a suspended state

**Cause**: insufficient quota in the cluster queue config, or the resource
flavor doesn't exist yet.

**Diagnosis**: the Ray cluster's head/worker pods stay suspended. Check
the `Workload` resource created alongside the `RayCluster` — its
`status.conditions.message` gives the reason, e.g.:
```
message: 'couldn''t assign flavors to pod set small-group-jobtest12:
insufficient quota for nvidia.com/gpu in flavor default-flavor in
ClusterQueue'
```

**Fix**:

1. Console → user's project → **Home → Search** → **Resources** →
   `ResourceFlavor` — create it if missing.
2. Check the cluster queue config against what the user's code requests.
3. Increase the quota if needed — see
   [[configuring-quota-management-for-distributed-workloads]].

## A user's Ray cluster is in a failed state

**Cause**: usually insufficient resources.

**Diagnosis**: head/worker pods not running. A newly-created Ray cluster
initially enters `failed` and this normally self-resolves once
reconciliation completes.

**Fix** (if it doesn't resolve): console → user's project → **Workloads →
Pods** → the pod → **Events** tab for the cause; also check
`status.conditions.message` on the `Workload` resource tied to the
`RayCluster`.

## A user's Ray cluster does not start

**Symptom**: after `cluster.apply()`, `cluster.details()`/`cluster.status()`
keeps reporting `Starting` instead of `Ready`; no pods are created.

**Diagnosis**: check `status.conditions.message` on both the `Workload`
resource and the `RayCluster` resource for the stall reason.

**Fix**:

1. Console → user's project → **Workloads → Pods**.
2. Confirm the KubeRay pod is `Running`; restart it if not.
3. Review the KubeRay pod's logs for errors.

## A user cannot create a Ray cluster or submit jobs

**Symptom**: after `cluster.apply()`:
```
RuntimeError: Failed to get RayCluster CustomResourceDefinition: (403)
Reason: Forbidden
... "rayclusters.ray.io is forbidden: User
\"system:serviceaccount:regularuser-project:regularuser-workbench\" cannot
list resource \"rayclusters\" in API group \"ray.io\" in the namespace
\"regularuser-project\""...
```

**Cause**: wrong/missing OpenShift login credentials in the notebook's
`TokenAuthentication` setup.

**Fix**:

1. Have the user get correct credentials: OpenShift console → username →
   **Copy login command** → log in as themselves in the new tab → **Display
   Token** → copy `token` and `server` from **Log in with this token**.
2. Set them in notebook code:
   ```python
   auth = TokenAuthentication(
       token = "<token>",
       server = "<server>",
       skip_tls=False
   )
   auth.login()
   ```
3. Confirm the user has correct permissions and is in the `rhods-users`
   group — see [[rhoai-user-types-and-permissions]].
