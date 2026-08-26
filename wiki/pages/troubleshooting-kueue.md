---
type: error
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1122-1226
verified: 2026-08-25
confidence: documented
---

# Troubleshooting Kueue workload management

Symptoms produced by [[kueue-workload-management]] /
[[configuring-kueue-workload-management]] when a workload interacts with
Kueue admission or queueing. If a problem isn't documented here or in the
release notes, contact Red Hat Support. See also
[[troubleshooting-distributed-workloads-administrators]] and
"Troubleshooting common problems with distributed workloads for users"
(not ingested).

## "failed to call webhook" error for Kueue

**Symptom**: after `cluster.apply()`, an `ApiException: (500)` whose body
includes:
```
failed calling webhook "mraycluster.kb.io": failed to call webhook: Post
"https://kueue-webhook-service.redhat-ods-applications.svc:443/mutate-ray-io-v1-raycluster?timeout=10s":
no endpoints available for service "kueue-webhook-service"
```

**Cause**: the Kueue pod isn't running.

**Diagnosis/fix**:

1. OpenShift console → user's project → **Workloads → Pods**.
2. Confirm the Kueue pod is `Running`; restart it if not.
3. Check its logs for the webhook server serving, e.g.:
   ```
   {"level":"info", ..., "msg":"Serving webhook server", "port":9443}
   ```

## "Default Local Queue …​ not found" error

**Symptom**: after `cluster.apply()`:
```
Default Local Queue with kueue.x-k8s.io/default-queue: true annotation not
found please create a default Local Queue or provide the local_queue name
in Cluster Configuration.
```

**Cause**: no default local queue is defined, and none was specified in
the cluster configuration.

**Diagnosis/fix**:

1. OpenShift console → user's project → **Home → Search** → **Resources**
   → `LocalQueue`. If none found, create one.
2. Provide the user the local queue details, and have them add it to their
   cluster configuration, or define a default local queue — see
   [[configuring-quota-management-for-distributed-workloads]].

## "local_queue provided does not exist" error

**Symptom**: after `cluster.apply()`:
```
local_queue provided does not exist or is not in this namespace. Please
provide the correct local_queue name in Cluster Configuration.
```

**Cause**: the cluster configuration's local queue name is wrong, or the
named queue exists in a different namespace than the one specified.

**Diagnosis/fix**:

1. Console → user's project → **Search** → `LocalQueue`.
   - If none found: create one.
   - If one or more found: give the user the correct name(s); confirm
     spelling and that `namespace` in their cluster config matches their
     project.
2. Alternatively, define a default local queue — see
   [[configuring-quota-management-for-distributed-workloads]].

## Pod terminated before its image finished pulling

**Symptom**: Kueue waits (5 minutes by default) for all workload pods to
become provisioned/running before marking the workload ready. A large
image still pulling after 5 minutes causes Kueue to fail the workload and
terminate its pods.

**Diagnosis**: console → user's project → **Workloads → Pods** → the pod →
**Events** tab → check whether the image pull completed.

**Fix** (if pulls regularly exceed 5 minutes):

- Add an `OnFailure` restart policy to Kueue-managed resources.
- Configure a custom `waitForPodsReady` timeout on the `Kueue` CR (lives in
  namespace `openshift-kueue-operator`, installed by the Red Hat build of
  Kueue Operator) — see Kueue's own "Enabling waitForPodsReady" docs
  (external, not ingested).
