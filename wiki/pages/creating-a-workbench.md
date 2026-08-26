---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md#L260-294
verified: 2026-08-25
confidence: documented
---

# Creating a workbench (RHOAI dashboard)

Creates a [[rhoai-workbench]] inside a [[rhoai-project]]. This page covers
the "Getting started" guide's version of this procedure; the dedicated
`raw/rhoai/3.5/creating-a-workbench.md` guide (4 units) has not been
ingested yet and likely has more detail — update this page rather than
duplicating it once that guide is ingested (CLAUDE.md rule 5).

## Preconditions

- Logged in to Red Hat OpenShift AI, project already created.
- If creating an S3 connection to storage outside RHOAI: endpoint URL,
  access key, secret key, region, bucket name for that storage.

## Steps

1. From the dashboard, **Projects** → select the target project → the
   project details page opens.
2. Click the **Workbenches** tab, then **Create workbench**.
3. Set the **Name** (display name).
4. Optional: **Edit resource name** — same character/length rules as
   projects; cannot change after creation.
5. Optional: set a **Description**.
6. **Workbench image**: pick from **Image selection** (see the image list on
   [[rhoai-workbench]]) — project-scoped images appear under a separate
   subheading from global images. Optionally view included packages via
   **View package information**. If multiple versions exist, pick one from
   **Version selection** (Red Hat recommends the most recent for latest
   packages). The image can be changed later.
7. **Deployment size**: if the (Technology Preview, as of 3.5) hardware
   profiles feature is off, pick a **Container size** (e.g. **Medium** for
   the example fine-tuning job in the distributed-workloads guide, not yet
   ingested). If hardware profiles are on, pick a **Hardware profile**
   instead (project-scoped ones appear under their own subheading); it sets
   guaranteed-minimum (request) and maximum (limit) CPU/memory, customizable
   via **Customize resource requests and limit**.
8. Optional: set **Environment variables** — for S3-compatible storage, set
   `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`; select **Secret** to
   store them as a Kubernetes Secret. Scope credentials to only the S3
   resources this project needs — sharing AWS credentials across projects
   risks compromised credentials and unauthorized access.
9. **Cluster storage**: either **Create new persistent storage** (retained
   after workbench shutdown — name, description, storage class [not
   changeable after creation], access mode if the storage class supports
   more than one, size in GiB/MiB) or **Use existing persistent storage**.
10. Optional: add a **Connection** — **Attach existing connections** to reuse
    one, or **Create connection** (pick a connection type; for S3-compatible
    object storage: name, optional description, access key, secret key,
    endpoint, region, bucket; for URI: name, optional description, URI).
11. Click **Create workbench**.

## Verification

The workbench appears on the project's **Workbenches** tab; associated
cluster storage appears on **Cluster storage**. **Status** shows **Starting**
then **Running**. Open the open-icon to launch the IDE in a new window.
