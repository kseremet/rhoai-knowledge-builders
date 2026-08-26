# Log

Append-only. One entry per operation, newest at the bottom.
Prefix format: `## [YYYY-MM-DD] <op> | <subject>` so it greps cleanly.

## [2026-08-24] ingest | gpu-operator ClusterPolicy CRD (nvidia.com_clusterpolicies.yaml, sections 1-4/7)

First ingest into an empty wiki. Source: `raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md`,
units `nvidiacom-clusterpoliciesyaml#1` through `#4` (of 7 total sections in
the source document; sections 5-7 not ingested this pass).

Created 12 pages, all type `component`, product `gpu-operator`, confidence
`documented`, no fixed version (source tracked at `ref: main`, rolling):
[[gpu-operator-clusterpolicy]] (parent CR, indexes the rest and documents the
`cdi`/`daemonsets`/`hostPaths` config blocks that aren't separate deployed
things), [[nvidia-driver]], [[nvidia-dcgm]], [[nvidia-dcgm-exporter]],
[[nvidia-device-plugin]], [[gpu-feature-discovery]], [[nvidia-mig-manager]]
(covers both `spec.mig` and `spec.migManager`), [[nvidia-gdrcopy]],
[[nvidia-gpudirect-storage]], [[nvidia-cc-manager]], [[kata-manager]],
[[kata-sandbox-device-plugin]].

Chose not to create separate pages for `cdi`, `daemonsets`, and `hostPaths` -
none of them is a thing deployed into the cluster, they're shared config
schemas/toggles that other components consume, so they're documented as
subsections of [[gpu-operator-clusterpolicy]] instead.

Cross-references added in both directions between the parent page and all 11
component pages, and between components that depend on each other (all depend
on [[nvidia-driver]]; [[nvidia-dcgm-exporter]] on [[nvidia-dcgm]];
[[nvidia-device-plugin]]/[[gpu-feature-discovery]]/[[nvidia-mig-manager]] on
each other via the MIG strategy and shared device-plugin configmap;
[[kata-manager]]/[[kata-sandbox-device-plugin]] on each other).

Noted as a finding worth a page reader's attention: `spec.kataManager` is
explicitly marked deprecated/ignored in the CRD schema while
`spec.kataSandboxDevicePlugin` carries no such notice - recorded on both
pages.

Flagged gap (see "Gaps" section on [[gpu-operator-clusterpolicy]]):
`spec.toolkit`, `spec.validator`, `spec.operator`, `spec.vgpuManager`,
`spec.vgpuDeviceManager`, `spec.vfioManager`, `spec.sandboxWorkloads`,
`spec.sandboxDevicePlugin`, `spec.psp`, `spec.psa`, `spec.nodeStatusExporter`,
and the CRD's `status` schema are all in the source document (sections 5-7/7)
but were not part of this ingest batch and are undocumented in this wiki.
`spec.toolkit` in particular is a significant omission since most components
above depend on the Container Toolkit at runtime.

rhkb done nvidiacom-clusterpoliciesyaml#1 nvidiacom-clusterpoliciesyaml#2 nvidiacom-clusterpoliciesyaml#3 nvidiacom-clusterpoliciesyaml#4

## [2026-08-25] project | ingest prioritization

User: main focus is Red Hat OpenShift AI; NVIDIA/gpu-operator docs are
relevant and stay priority; deprioritize OpenShift (`ocp`) docs except
monitoring, service mesh, and authentication/authorization.

Trimmed `sources.yaml`'s `ocp.include` to `monitoring`,
`observability_overview`, `authentication_and_authorization`,
`security_and_compliance`, `service_mesh` (slug unverified - needs
`rhkb guides --source ocp --yaml` with network access to confirm). Marked
564 units from the other 12 already-fetched OCP guides (ai-workloads,
extensions, hardware-accelerators, ingress-and-load-balancing,
machine-management, networking-overview, nodes, operators,
postinstallation-configuration, scalability-and-performance,
specialized-hardware-and-driver-enablement, storage) as `skipped` in
`state.json` rather than deleting the fetched `raw/` files - reversible via
`rhkb reset`. `rhkb next` outstanding count for `ocp` dropped from 743 to
179. Went with `rhkb skip` + a trimmed `include:` over disabling the `ocp`
source outright, so the guides actually wanted still surface normally.

## [2026-08-25] ingest | gpu-operator ClusterPolicy CRD completion + GPUCluster CRD start

Source: `raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md`,
units `nvidiacom-clusterpoliciesyaml#5` through `#7` (completes that
document, 7/7 sections now ingested) plus
`raw/gpu-operator/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml.md`,
unit `nvidiacom-gpuclustersyaml#1` (starts a new document, 1 of 3 sections).

Updated [[gpu-operator-clusterpolicy]]: removed the old "Gaps" section (now
closed), added the `psa`/`psp`/`sandboxWorkloads` config-block notes and the
`status` schema section, expanded the managed-components list, noted
`spec.daemonsets`/`spec.dcgm`/`spec.devicePlugin`/`spec.driver`/`spec.gfd`/
`spec.nodeStatusExporter`/`spec.operator`/`spec.toolkit` are the fields
`ClusterPolicy.spec` requires, and added a pointer to the new
[[gpu-operator-gpucluster]] page so the two CRDs aren't conflated.

Created 8 new component pages: [[nvidia-container-toolkit]],
[[node-status-exporter]], [[gpu-operator-controller]] (`spec.operator` - the
operator's own controller-manager pod, not an operand),
[[gpu-operator-validator]], [[nvidia-sandbox-device-plugin]] (`spec.sandboxDevicePlugin`
- confirmed distinct from the existing [[kata-sandbox-device-plugin]] page,
cross-linked both ways to prevent conflating them), [[nvidia-vfio-manager]],
[[nvidia-vgpu-manager]], [[nvidia-vgpu-device-manager]].

Created [[gpu-operator-gpucluster]] for the new `GPUCluster` CRD
(`v1alpha1`, DRA-based) - marked explicitly partial (1 of 3 source sections)
with an "Ingest status" note so a reader doesn't take field absence as
evidence of non-existence. Recorded its `spec.daemonsets` as identical to
`ClusterPolicy`'s. Recorded its `spec.dcgm` as only partially read but
flagged a real behavioral difference already visible in the schema
description (standalone DCGM hostengine opt-in there vs. no such note on
`ClusterPolicy`) and cross-linked that finding from [[nvidia-dcgm]] so a
reader of the `ClusterPolicy` page sees the caveat too.

Updated [[nvidia-driver]]: linked the newly-created [[nvidia-container-toolkit]]
and [[gpu-operator-validator]]; noted the `driverManager`/`manager` initContainer
pattern is duplicated independently three times across [[nvidia-driver]],
[[nvidia-vfio-manager]], and [[nvidia-vgpu-manager]] (not shared).

rhkb done nvidiacom-clusterpoliciesyaml#5 nvidiacom-clusterpoliciesyaml#6 nvidiacom-clusterpoliciesyaml#7 nvidiacom-gpuclustersyaml#1

## [2026-08-25] ingest | RHOAI 3.5 - Getting started guide

First RHOAI-source ingest (previous ingests were all the upstream
gpu-operator CRDs). User chose to start `rhoai` with the foundational
install/getting-started guides rather than whatever `rhkb next` would emit
alphabetically (which was "Building RAG applications with OGX"). Source:
`raw/rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md`,
all 7 units (full guide) - selected manually via a scoped `iter_units` call
rather than `rhkb next`, since `rhkb` has no per-document filter, only
per-source/tier.

Created 8 pages, product `rhoai`, version `rhoai-3.5`, confidence
`documented`: [[red-hat-openshift-ai]] (concept - platform overview, the
data science workflow, an index of where things live and what's not ingested
yet), [[rhoai-glossary]] (concept - full term glossary from the guide, with
[[wiki-link]]s out to pages that already cover a term properly and notes on
which source guide will supersede a term-only definition),
[[rhoai-project]] + [[creating-a-rhoai-project]] (component + procedure),
[[rhoai-workbench]] + [[creating-a-workbench]] + [[building-rstudio-workbench-images]]
(component + two procedures - workbench creation and the separate technical
step of building the Technology Preview RStudio images), and
[[logging-in-to-openshift-ai]] (procedure).

Flagged rather than guessed: `raw/rhoai/3.5/creating-a-workbench.md` is a
separate, not-yet-ingested guide that likely has deeper workbench-creation
detail than this getting-started chapter - noted on [[creating-a-workbench]]
so a future ingest updates that page instead of creating a duplicate
(CLAUDE.md rule 5). Several [[rhoai-glossary]] terms (AI pipelines,
distributed workloads, model catalog, model registry) point to their own
not-yet-ingested guides for the same reason - the glossary page gives the
short definition only, not the full component treatment.

rhkb done rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:11-data-science-workflowcopy-linklink-copied-to-clipboard rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:13-glossary-of-common-termscopy-linklink-copied-to-clipboard rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:chapter-2-logging-in-to-openshift-aicopy-linklink-copied-to- rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:41-about-workbench-imagescopy-linklink-copied-to-clipboard rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:42-building-the-rstudio-server-workbench-imagescopy-linklink rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:43-creating-a-workbenchcopy-linklink-copied-to-clipboard rhoai/3.5/getting-started-with-red-hat-openshift-ai-self-managed.md:chapter-5-next-stepscopy-linklink-copied-to-clipboard

## [2026-08-25] ingest | 7 unit(s)

- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 1. OverviewCopy linkLink copied to clipboard! > 1.1. Data science workflowCopy linkLink copied to clipboard!
- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 1. OverviewCopy linkLink copied to clipboard! > 1.3. Glossary of common termsCopy linkLink copied to clipboard!
- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 2. Logging in to OpenShift AICopy linkLink copied to clipboard!
- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 4. Creating a workbench and selecting an IDECopy linkLink copied to clipboard! > 4.1. About workbench imagesCopy linkLink copied to clipboard!
- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 4. Creating a workbench and selecting an IDECopy linkLink copied to clipboard! > 4.2. Building the RStudio Server workbench imagesCopy linkLink copied to clipboard!
- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 4. Creating a workbench and selecting an IDECopy linkLink copied to clipboard! > 4.3. Creating a workbenchCopy linkLink copied to clipboard!
- Getting started with Red Hat OpenShift AI Self-Managed > Chapter 5. Next stepsCopy linkLink copied to clipboard!

## [2026-08-25] ingest | Installing and uninstalling OpenShift AI Self-Managed (RHOAI 3.5)

Ingested all 20 units of `brief-install.md` (RHOAI 3.5 "Installing and
uninstalling OpenShift AI Self-Managed" guide) into 27 new wiki pages: 4
component (`rhoai-operator`, `datasciencecluster`, `dscinitialization`,
`ogx-server`), 3 concept (`rhoai-operator-update-channels`,
`how-rhoai-handles-certificates`, `ogx-server-ca-bundle-reference`), 18
procedure, and 2 error pages. Also updated the existing `red-hat-openshift-ai`
and `logging-in-to-openshift-ai` pages, which previously pointed at this
guide as "not yet ingested" — they now link into the new pages.

**Splitting decisions**: Chapter 8 (certificates, the largest chapter) was
split into a concept page explaining the `odh-trusted-ca-bundle` mechanism
plus nine procedure pages, one per distinct configuration target
(cluster-wide bundle, custom bundle, S3/object-storage access, pipelines,
workbenches, model serving, OGX, unmanaging, removing) rather than one
mega-page, since each has its own preconditions/steps/verification a reader
would want in isolation. `OGXServer`'s CA-bundle reference material
(subfields, limits, validation/failure conditions, client usage) was kept
separate from the `ogx-server` component page and the
`configuring-ca-bundle-for-ogx-server` procedure — it's genuinely reference
material, not "what is this thing" or "do this task." Chapter 10
(troubleshooting) became one `error` page with 8 symptom sub-sections rather
than 8 separate pages: they share one diagnostic entry point (the
`rhods-operator` pod logs) and are practically a single runbook a reader
would open together; the dashboard-inaccessible symptom (10.6) was kept as
its own page per the brief's explicit callout and because it's reached via
a different diagnostic path (Workloads → Pods → All Projects, not the
operator pod specifically).

**Not created / deferred**: no separate pages for `feastoperator`,
`mlflowoperator`, `trustyai`, or model registry beyond mentioning them as
`DataScienceCluster` component keys — the guide gives no configuration
detail for them beyond `managementState`. Component-specific accelerator
enablement (NVIDIA/AMD/Gaudi/Spyre) and the disconnected-environment install
variant were left as forward references — this guide only points at them,
it doesn't contain their content.

**Cross-references added**: new pages link into `rhoai-workbench`,
`red-hat-openshift-ai`, `logging-in-to-openshift-ai`, and the existing
`gpu-operator-clusterpolicy` family (as the upstream CRDs that
`enabling-accelerators` and `ogx-server`'s GPU-node precondition depend on).

**Findings / gaps worth flagging**:

1. The drafting pass didn't have the raw source file available and initially
   cited heading anchors reconstructed from the brief's embedded URLs.
   Fetched
   `raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md`
   directly afterward and replaced every citation across these 27 pages
   with real `#L<start>-<end>` line ranges against the actual headings
   (`grep -n` of the guide's `##`/`###`/`####` structure), per CLAUDE.md's
   line-range citation style. `datasciencecluster` and
   `installing-rhoai-components` cite the whole `## Chapter 3.4` block
   (L342-509) rather than a narrower range since the CR is introduced and
   used continuously across its CLI/web-console/status subsections;
   `configuring-kserve-oauth-proxy-sidecar-resources` was re-pointed at its
   actual subsection (L393-421) after confirming the content's real
   location by grepping for `oauthProxy` in the raw file, rather than the
   generic anchor the drafting pass had guessed.
2. Chapter 10 (troubleshooting) documents 5 of its 8 symptoms as occurring
   "in unknown circumstances" with resolution = "contact Red Hat support" —
   this is a real gap in Red Hat's own source material, not an omission
   here; noted explicitly on `troubleshooting-rhoai-operator-installation`.
3. `rhoai-dashboard-not-accessible` (10.6) has no documented root-cause
   list at all, unlike the rest of chapter 10 — the guide gives only a
   generic "find the erroring pod, read its logs" procedure. Flagged
   on-page rather than inventing causes (CLAUDE.md rule 1).
4. `using-cluster-wide-certificate-for-model-serving` (8.5.4) has no
   verification section in the source guide — flagged on-page.
5. Several components referenced as install prerequisites are named but not
   themselves documented in this guide (cert-manager Operator, Red Hat
   build of Kueue Operator, Red Hat Connectivity Link Operator, Leader
   Worker Set Operator, Service Mesh Operator 3.x, NFD/KMM) — left as plain
   external mentions rather than wiki-links, since no page exists for them
   yet and this guide gives no detail to write one from.
6. OKE's licensing-exception mechanism for dependency Operators (3.1.1) is
   named but not explained procedurally — flagged as a gap on
   `installing-the-rhoai-operator`.

## [2026-08-25] ingest | 20 unit(s)

- Installing and uninstalling OpenShift AI Self-Managed > Chapter 1. Architecture of OpenShift AI Self-ManagedCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 3. Installing and deploying OpenShift AICopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 3. Installing and deploying OpenShift AICopy linkLink copied to clipboard! > 3.1. Requirements for OpenShift AI Self-ManagedCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 3. Installing and deploying OpenShift AICopy linkLink copied to clipboard! > 3.2. Configuring custom namespacesCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 3. Installing and deploying OpenShift AICopy linkLink copied to clipboard! > 3.3. Installing the Red Hat OpenShift AI OperatorCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 3. Installing and deploying OpenShift AICopy linkLink copied to clipboard! > 3.4. Installing and managing Red Hat OpenShift AI componentsCopy linkLink copied to clipboard! (1/2)
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 3. Installing and deploying OpenShift AICopy linkLink copied to clipboard! > 3.4. Installing and managing Red Hat OpenShift AI componentsCopy linkLink copied to clipboard! (2/2)
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 4. Configuring pipelines with your own Argo Workflows instanceCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 5. Installing the distributed workloads componentsCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 6. Accessing the dashboardCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 8. Working with certificatesCopy linkLink copied to clipboard! > 8.1. Understanding how OpenShift AI handles certificatesCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed > Chapter 8. Working with certificatesCopy linkLink copied to clipboard! > 8.4. Adding certificates to a custom CA bundleCopy linkLink copied to clipboard!

## [2026-08-25] ingest | Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment (rhoai-3.5)

Source: `raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed-in-a-d.md`
(34 units). This guide is structurally near-identical to the connected-cluster
"Installing and uninstalling OpenShift AI Self-Managed" guide already ingested
(27 pages) — same chapters, same CR objects, same certificate/troubleshooting
content — so this ingest focused on finding and isolating the genuinely
disconnected-specific differences rather than re-ingesting shared content.

**Created (1 new page)**

- `mirroring-images-for-disconnected-install` (procedure) — Chapter 3.2. This
  is the one section of the guide with no connected-cluster equivalent at all:
  using oc-mirror v2 to mirror RHOAI's required images to a private registry
  before the Operator can be installed. Linked bidirectionally with
  `installing-the-rhoai-operator` (next-step link forward, disconnected-note
  link back).

**Updated (1 existing page)**

- `installing-the-rhoai-operator` — added an "In a disconnected environment"
  section to the Preconditions, plus an inline comment on the `Subscription`
  YAML example, covering the three real differences found in Chapter 3.1/3.4
  of the disconnected guide:
  1. The internet-access precondition applies to the *mirroring machine*, not
     the cluster, in a disconnected install.
  2. An extra precondition: images must be mirrored first (links to the new
     `mirroring-images-for-disconnected-install` page).
  3. After cluster install, the Cluster Samples Operator must be configured
     for a restricted network (external doc, not ingested).
  4. The `Subscription.spec.source` field differs: `cs-redhat-operator-index`
     (the mirror-generated CatalogSource) instead of the connected default
     `redhat-operators`. This was the most concrete, easy-to-miss difference
     — same CR shape, different field value, exactly what rule 2 (never quote
     a moving value inline without version-qualifying) is meant to catch.
  All other content in the CLI/web-console steps, verification, and next
  steps was identical between the two guides and was left untouched.

**Deliberately NOT touched / NOT duplicated** — checked against the connected
guide's existing pages and found no meaningful difference, or the page exists
in the index under a name not available for direct diffing during this
session (left untouched per rule 5 rather than risking a duplicate):

- Preface / Chapter 1 (Architecture) — identical to `rhoai-operator`'s
  existing "Namespaces it creates" content. No change.
- Chapter 2 (Understanding update channels) — identical to
  `rhoai-operator-update-channels` (same channel table, same notes on RHOAI
  3.0/`fast-3.x`, `embedded`/`beta` deprecation, sequential-upgrade behavior).
  No change.
- 3.1 Requirements — the disconnected-specific bullets (image mirroring,
  mirroring-machine internet access, Cluster Samples Operator) were folded
  into `installing-the-rhoai-operator`'s new disconnected note (see above).
  The rest (subscriptions, cluster-admin access, supported OpenShift
  versions, cluster sizing, storage, identity provider, custom namespaces) is
  identical to what's already documented there.
- 3.1.2 Component requirements — generic component-prerequisite content, not
  disconnected-specific; out of scope for this ingest, not created as a page.
- 3.3 Configuring custom namespaces — identical to
  `configuring-custom-namespaces-for-rhoai`. No change.
- 3.4 Installing the Operator (web console path) — identical aside from the
  CLI precondition/source differences already folded in above.
- 3.5 Installing and managing RHOAI components (incl. the KServe OAuth-proxy-
  sidecar procedure and "Viewing installed components") — identical to
  `installing-rhoai-components`. The OAuth-sidecar and "viewing installed
  components" content maps to `configuring-kserve-oauth-proxy-sidecar-
  resources` and `logging-in-to-openshift-ai` respectively — both already
  exist but were not available to diff directly this session; left untouched.
- Chapter 4 (Own Argo Workflows instance) — identical to
  `configuring-pipelines-with-own-argo-workflows`. No change.
- Chapter 5 (Distributed workloads components) — identical to
  `installing-distributed-workloads-components`, aside from cross-reference
  URLs pointing at the disconnected guide's own chapter numbers. No change.
- Chapter 6 (Accessing the dashboard) — identical to
  `accessing-the-rhoai-dashboard`, including the 3.4 EA1 `rh-ai` URL note.
  No change.
- Chapter 7 (Enabling accelerators) — identical to `enabling-accelerators`.
  No change.
- Chapter 8 (Working with certificates), all of 8.1–8.7 — identical to the
  existing certificate pages: `how-rhoai-handles-certificates`,
  `adding-certificates-to-cluster-wide-ca-bundle`,
  `adding-certificates-to-custom-ca-bundle`,
  `using-cluster-wide-certificate-for-model-serving`,
  `managing-certificates-without-the-rhoai-operator`, and
  `removing-the-ca-bundle` — same steps, same commands, same YAML. No
  changes. Sub-sections 8.5.1/8.5.2/8.5.3/8.5.5–8.5.7 map to
  `configuring-certificates-for-s3-object-storage-access`,
  `configuring-certificate-for-pipelines`,
  `configuring-certificate-for-workbenches`,
  `configuring-ca-bundle-for-ogx-server`, and `ogx-server-ca-bundle-reference`
  per the index — already exist, not available to diff this session, left
  untouched rather than guessed at.
- Chapter 9 (Logs and audit records) — identical to
  `configuring-the-rhoai-operator-logger` and `viewing-rhoai-audit-records`.
  No change.
- Chapter 10 (Troubleshooting), all 9 symptoms — **notable negative
  finding.** The two symptoms suspected to be disconnected-specific (10.2
  "cannot be retrieved from the image registry", 10.9 "ODH parameter secret
  does not get created") turned out to be **byte-for-byte identical** to
  symptoms already documented in `troubleshooting-rhoai-operator-
  installation` (the `Failure to pull from quay` symptom, and the `ERROR:
  Addon managed odh parameter secret does not exist.` symptom respectively)
  — same problem text, same diagnosis, same "contact Red Hat support"
  resolution. Neither is actually disconnected-specific; the guide just
  renumbers the same troubleshooting chapter. No new error page created. The
  remaining 7 symptoms are likewise identical to existing content (10.6 maps
  to the separate `rhoai-dashboard-not-accessible` page).
- Chapter 11 (Uninstalling) — identical to `uninstalling-rhoai`. No change.
- Legal Notice — boilerplate.

**Gaps / findings worth flagging**

- Chapter 10 of the disconnected guide contributes *no* new troubleshooting
  content — only a renumbering of the same 9 symptoms already in
  `troubleshooting-rhoai-operator-installation`.
- Eight pages the index lists as already existing
  (`configuring-kserve-oauth-proxy-sidecar-resources`,
  `logging-in-to-openshift-ai`,
  `configuring-certificates-for-s3-object-storage-access`,
  `configuring-certificate-for-pipelines`,
  `configuring-certificate-for-workbenches`,
  `configuring-ca-bundle-for-ogx-server`, `ogx-server-ca-bundle-reference`,
  `rhoai-dashboard-not-accessible`) map to sections of this disconnected
  guide but weren't available to diff directly during this ingest session.
  Spot-checking the disconnected guide's prose for these sections found
  nothing disconnected-specific, so an update is likely unneeded — but this
  should be confirmed directly against those pages' content in a follow-up
  pass rather than taken on faith here.

## [2026-08-25] ingest | 20 unit(s)

- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.1. Requirements for OpenShift AI Self-ManagedCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.2. Mirroring images to a private registry for a disconnected installationCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.3. Configuring custom namespacesCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.4. Installing the Red Hat OpenShift AI OperatorCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.5. Installing and managing Red Hat OpenShift AI componentsCopy linkLink copied to clipboard! (1/2)
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.5. Installing and managing Red Hat OpenShift AI componentsCopy linkLink copied to clipboard! (2/2)
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 4. Configuring pipelines with your own Argo Workflows instanceCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 5. Installing the distributed workloads componentsCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 6. Accessing the dashboardCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 8. Working with certificatesCopy linkLink copied to clipboard! > 8.1. Understanding how OpenShift AI handles certificatesCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 8. Working with certificatesCopy linkLink copied to clipboard! > 8.4. Adding certificates to a custom CA bundleCopy linkLink copied to clipboard!

## [2026-08-25] ingest | 21 unit(s)

- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 1. Architecture of OpenShift AI Self-ManagedCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.1. Requirements for OpenShift AI Self-ManagedCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.2. Mirroring images to a private registry for a disconnected installationCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.3. Configuring custom namespacesCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.4. Installing the Red Hat OpenShift AI OperatorCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.5. Installing and managing Red Hat OpenShift AI componentsCopy linkLink copied to clipboard! (1/2)
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 3. Deploying OpenShift AI in a disconnected environmentCopy linkLink copied to clipboard! > 3.5. Installing and managing Red Hat OpenShift AI componentsCopy linkLink copied to clipboard! (2/2)
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 4. Configuring pipelines with your own Argo Workflows instanceCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 5. Installing the distributed workloads componentsCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 6. Accessing the dashboardCopy linkLink copied to clipboard!
- Installing and uninstalling OpenShift AI Self-Managed in a disconnected environment > Chapter 8. Working with certificatesCopy linkLink copied to clipboard! > 8.1. Understanding how OpenShift AI handles certificatesCopy linkLink copied to clipboard!

## [2026-08-25] ingest | raw/rhoai/3.5/managing-openshift-ai.md

Ingested all 29 units of RHOAI 3.5's "Managing OpenShift AI" guide (day-2
operations, sibling to the two already-ingested "installing" guides).

### Created (31 new pages)

**Ch1 users/groups**: [[rhoai-user-types-and-permissions]] (concept),
[[managing-rhoai-user-groups]], [[deleting-rhoai-users]].

**Ch2 custom workbench images**: [[creating-custom-workbench-images]] (2.1
build-from-default + 2.2 build-your-own guidelines merged — they're two
halves of one build task), [[importing-custom-workbench-images]] (2.3
enabling the menu + 2.4 the import wizard — one continuous admin flow).
Confirmed genuinely new material, not overlapping [[rhoai-workbench]] or
[[creating-a-workbench]] (those cover *using* provided images, not
*building* custom ones).

**Ch3 dashboard applications**: [[managing-dashboard-applications]] — kept
3.1-3.5 as one page (all short, all edit the same two CRs —
`OdhApplication`/`OdhDashboardConfig` — for closely related dashboard-tile
concerns) rather than five stub pages.

**Ch4-6**: [[creating-project-scoped-resources]],
[[allocating-additional-resources-to-rhoai]] (kept despite being a
one-paragraph forward reference — it has a real citable claim and a
distinct topic, so cite-or-delete says keep a thin page rather than drop
it or bury it in another page's margins),
[[customizing-component-deployment-resources]] (6.1-6.4 merged: one
resource-customization mechanism with enable/disable/re-enable as its three
states).

**Ch7 accelerators — split per vendor**: [[enabling-nvidia-gpus]],
[[enabling-intel-gaudi-accelerators]], [[enabling-amd-gpus]]. Splitting
decision: each vendor's procedure is substantial (its own prerequisites,
its own Operator, its own gotchas — Gaudi's PID-limit tuning, AMD's
`lspci`-based verification sub-procedure) and independently actionable
without the others, matching the install guide's own certificate-chapter
precedent for splitting a meaty chapter along its natural seams (there:
per-consumer; here: per-vendor). A shared vendor-agnostic step
("create a hardware profile") stays a forward reference in each, since the
hardware-profiles guide itself isn't ingested yet.

**Ch8 Kueue — split by concept/procedure/error**:
[[kueue-workload-management]] (concept: states, enforcement, restrictions,
cross-role workflow), [[configuring-kueue-workload-management]] (procedure:
install/activate/dashboard-enable), [[troubleshooting-kueue]] (error, 4
symptoms), [[migrating-to-red-hat-build-of-kueue-operator]] (procedure,
kept separate from "configuring" since it's a one-time transition with its
own preconditions gate, not initial setup).

**Ch9 distributed workloads**:
[[configuring-quota-management-for-distributed-workloads]] (9.1 procedure
+ 9.2 example configs merged — the examples are literally worked instances
of 9.1's own YAML shapes), [[configuring-cluster-for-rdma]],
[[troubleshooting-distributed-workloads-administrators]] (error, 4
symptoms).

**Ch10 centralized auth — split by concept/procedure/error**:
[[centralized-authentication-service]] (concept),
[[configuring-oidc-centralized-authentication]] (10.2, includes its
IdP-group-to-ClusterRole "next steps" since that's required for the OIDC
path to actually grant access),
[[configuring-service-account-token-authentication]] (10.3, the separate
programmatic-access path), [[configuring-custom-ca-for-oidc]] (10.4, short
but a distinct precondition-gated task), and
[[troubleshooting-centralized-authentication-service]] (error, 7
symptoms). Same split rationale as Kueue and accelerators: each piece is
independently actionable and has its own preconditions.

**Ch11 backing up**: [[backing-up-rhoai-data]] (11.1 storage + 11.2
cluster backup — two short, closely-related backup concerns on one page).

**Ch12 observability — one page per numbered section** (all Technology
Preview, as of 3.5, each independently actionable and gated on the
previous): [[enabling-observability-stack]],
[[enabling-observability-dashboard]],
[[collecting-metrics-from-user-workloads]],
[[exporting-metrics-to-external-tools]],
[[viewing-traces-in-external-tools]], [[accessing-rhoai-alerts]].

**Ch13**: no new pages — see "Updated" below, both sections turned out to
duplicate already-ingested content.

### Updated (6 pages)

- [[enabling-accelerators]] — was a pure forward-reference stub citing only
  the install guide; rewritten as a real index page linking the three new
  vendor procedure pages, since this guide's Chapter 7 turned out to be
  exactly the content it was forward-referencing. IBM Spyre remains an
  explicit gap (still not ingested from any guide).
- [[rhoai-workbench]] — its "build a custom image" forward reference now
  points at [[creating-custom-workbench-images]] /
  [[importing-custom-workbench-images]]; added a ROCm→[[enabling-amd-gpus]]
  cross-link.
- [[datasciencecluster]] — the `kueue: Unmanaged` bullet now links to
  [[kueue-workload-management]] and
  [[migrating-to-red-hat-build-of-kueue-operator]] in addition to the
  existing install-guide link.
- [[installing-distributed-workloads-components]] — its "Next step" forward
  reference to "Managing distributed workloads" (previously "not yet
  ingested") now links [[kueue-workload-management]],
  [[configuring-kueue-workload-management]], and
  [[configuring-quota-management-for-distributed-workloads]]; also linked
  its GPU precondition bullet to the new
  [[enabling-nvidia-gpus]]/[[enabling-amd-gpus]] pages and its RDMA mention
  to [[configuring-cluster-for-rdma]].
- [[configuring-the-rhoai-operator-logger]] — Chapter 13.1/13.1.1 turned
  out to be a **verbatim duplicate** of the already-ingested install-guide
  content (same log-level table, same `oc patch` command, same log-streaming
  snippet). Added this guide's line range as a second `sources` entry and a
  note explaining the duplication; no content changed, and no new page was
  created (rule 5).
- [[viewing-rhoai-audit-records]] — same situation as above: Chapter 13.2 is
  a verbatim duplicate of the install-guide procedure (identical
  `oc adm node-logs` / `jq` commands). Added the second source citation and
  a duplication note; no content changed.

### Deliberately not touched / gaps

- **`working_with_accelerators` (hardware profiles) guide** — every vendor
  accelerator page (NVIDIA/Gaudi/AMD) and the Kueue chapter all forward-
  reference "Working with hardware profiles" as the shared next step. Not
  ingested; noted as a gap on every page that references it.
- **IBM Spyre accelerators** — mentioned only in the original
  [[enabling-accelerators]] stub; this guide's Chapter 7 doesn't cover it
  either. Still an open gap.
- **Kueue CRD component pages** (`ResourceFlavor`, `ClusterQueue`,
  `LocalQueue`, the `Kueue` CR itself) — described richly across Ch8/Ch9 but
  not split into their own `component` pages (only [[datasciencecluster]]
  and [[dscinitialization]] exist as CR component pages today). Given the
  volume already ingested this round, treating these as a candidate for a
  future ingest pass rather than adding 4 more pages now — flagging instead
  of silently skipping.
- **`GatewayConfig` / Gateway API objects** — same call as Kueue CRDs:
  richly described procedurally (Ch10) but not split into a `component`
  page. Candidate for a future pass.
- Did not touch [[rhoai-dashboard-not-accessible]] despite
  [[troubleshooting-centralized-authentication-service]] covering a
  similarly-named 403 symptom — confirmed they're genuinely different
  failure modes (generic post-install dashboard-unreachable vs.
  OIDC-authenticated-but-unauthorized) and cross-linked them instead of
  merging.
- "Deploying models" / KServe serving-runtime details referenced from
  [[creating-project-scoped-resources]] — out of scope for this guide, not
  ingested.
- OpenShift-side docs referenced throughout (OADP backup/restore, audit-log
  policy config, SR-IOV Operator install, machine-set scaling, etc.) are
  left as named-but-not-ingested per existing wiki convention.

### Findings

- Two whole sections of Chapter 13 (13.1 Operator logger, 13.2 audit
  records) are word-for-word duplicates of already-ingested install-guide
  content — the guide's own text literally repeats itself across manuals.
  Worth knowing for future ingests: check overlap at the *section* level,
  not just the *page* level, since RHOAI's docs set reuses content across
  guides.
- Kueue's `managementState` story changed meaningfully since the install
  guide was ingested: **as of 2.24**, embedded Kueue (`Managed`) is
  deprecated in favor of the Red Hat build of Kueue Operator (`Unmanaged`).
  The existing [[installing-distributed-workloads-components]] page still
  correctly showed `Unmanaged` as the only supported value, so no
  contradiction to record on a version page — but
  [[migrating-to-red-hat-build-of-kueue-operator]] documents the
  transition for anyone still on the deprecated path.

## [2026-08-25] ingest | raw/rhoai/3.5/managing-openshift-ai.md

Ingested all 29 units of RHOAI 3.5's "Managing OpenShift AI" guide (day-2
operations, sibling to the two already-ingested "installing" guides).

### Created (31 new pages)

**Ch1 users/groups**: [[rhoai-user-types-and-permissions]] (concept),
[[managing-rhoai-user-groups]], [[deleting-rhoai-users]].

**Ch2 custom workbench images**: [[creating-custom-workbench-images]] (2.1
build-from-default + 2.2 build-your-own guidelines merged — they're two
halves of one build task), [[importing-custom-workbench-images]] (2.3
enabling the menu + 2.4 the import wizard — one continuous admin flow).
Confirmed genuinely new material, not overlapping [[rhoai-workbench]] or
[[creating-a-workbench]] (those cover *using* provided images, not
*building* custom ones).

**Ch3 dashboard applications**: [[managing-dashboard-applications]] — kept
3.1-3.5 as one page (all short, all edit the same two CRs —
`OdhApplication`/`OdhDashboardConfig` — for closely related dashboard-tile
concerns) rather than five stub pages.

**Ch4-6**: [[creating-project-scoped-resources]],
[[allocating-additional-resources-to-rhoai]] (kept despite being a
one-paragraph forward reference — it has a real citable claim and a
distinct topic, so cite-or-delete says keep a thin page rather than drop
it or bury it in another page's margins),
[[customizing-component-deployment-resources]] (6.1-6.4 merged: one
resource-customization mechanism with enable/disable/re-enable as its three
states).

**Ch7 accelerators — split per vendor**: [[enabling-nvidia-gpus]],
[[enabling-intel-gaudi-accelerators]], [[enabling-amd-gpus]]. Splitting
decision: each vendor's procedure is substantial (its own prerequisites,
its own Operator, its own gotchas — Gaudi's PID-limit tuning, AMD's
`lspci`-based verification sub-procedure) and independently actionable
without the others, matching the install guide's own certificate-chapter
precedent for splitting a meaty chapter along its natural seams (there:
per-consumer; here: per-vendor). A shared vendor-agnostic step
("create a hardware profile") stays a forward reference in each, since the
hardware-profiles guide itself isn't ingested yet.

**Ch8 Kueue — split by concept/procedure/error**:
[[kueue-workload-management]] (concept: states, enforcement, restrictions,
cross-role workflow), [[configuring-kueue-workload-management]] (procedure:
install/activate/dashboard-enable), [[troubleshooting-kueue]] (error, 4
symptoms), [[migrating-to-red-hat-build-of-kueue-operator]] (procedure,
kept separate from "configuring" since it's a one-time transition with its
own preconditions gate, not initial setup).

**Ch9 distributed workloads**:
[[configuring-quota-management-for-distributed-workloads]] (9.1 procedure
+ 9.2 example configs merged — the examples are literally worked instances
of 9.1's own YAML shapes), [[configuring-cluster-for-rdma]],
[[troubleshooting-distributed-workloads-administrators]] (error, 4
symptoms).

**Ch10 centralized auth — split by concept/procedure/error**:
[[centralized-authentication-service]] (concept),
[[configuring-oidc-centralized-authentication]] (10.2, includes its
IdP-group-to-ClusterRole "next steps" since that's required for the OIDC
path to actually grant access),
[[configuring-service-account-token-authentication]] (10.3, the separate
programmatic-access path), [[configuring-custom-ca-for-oidc]] (10.4, short
but a distinct precondition-gated task), and
[[troubleshooting-centralized-authentication-service]] (error, 7
symptoms). Same split rationale as Kueue and accelerators: each piece is
independently actionable and has its own preconditions.

**Ch11 backing up**: [[backing-up-rhoai-data]] (11.1 storage + 11.2
cluster backup — two short, closely-related backup concerns on one page).

**Ch12 observability — one page per numbered section** (all Technology
Preview, as of 3.5, each independently actionable and gated on the
previous): [[enabling-observability-stack]],
[[enabling-observability-dashboard]],
[[collecting-metrics-from-user-workloads]],
[[exporting-metrics-to-external-tools]],
[[viewing-traces-in-external-tools]], [[accessing-rhoai-alerts]].

**Ch13**: no new pages — see "Updated" below, both sections turned out to
duplicate already-ingested content.

### Updated (6 pages)

- [[enabling-accelerators]] — was a pure forward-reference stub citing only
  the install guide; rewritten as a real index page linking the three new
  vendor procedure pages, since this guide's Chapter 7 turned out to be
  exactly the content it was forward-referencing. IBM Spyre remains an
  explicit gap (still not ingested from any guide).
- [[rhoai-workbench]] — its "build a custom image" forward reference now
  points at [[creating-custom-workbench-images]] /
  [[importing-custom-workbench-images]]; added a ROCm→[[enabling-amd-gpus]]
  cross-link.
- [[datasciencecluster]] — the `kueue: Unmanaged` bullet now links to
  [[kueue-workload-management]] and
  [[migrating-to-red-hat-build-of-kueue-operator]] in addition to the
  existing install-guide link.
- [[installing-distributed-workloads-components]] — its "Next step" forward
  reference to "Managing distributed workloads" (previously "not yet
  ingested") now links [[kueue-workload-management]],
  [[configuring-kueue-workload-management]], and
  [[configuring-quota-management-for-distributed-workloads]]; also linked
  its GPU precondition bullet to the new
  [[enabling-nvidia-gpus]]/[[enabling-amd-gpus]] pages and its RDMA mention
  to [[configuring-cluster-for-rdma]].
- [[configuring-the-rhoai-operator-logger]] — Chapter 13.1/13.1.1 turned
  out to be a **verbatim duplicate** of the already-ingested install-guide
  content (same log-level table, same `oc patch` command, same log-streaming
  snippet). Added this guide's line range as a second `sources` entry and a
  note explaining the duplication; no content changed, and no new page was
  created (rule 5).
- [[viewing-rhoai-audit-records]] — same situation as above: Chapter 13.2 is
  a verbatim duplicate of the install-guide procedure (identical
  `oc adm node-logs` / `jq` commands). Added the second source citation and
  a duplication note; no content changed.

### Deliberately not touched / gaps

- **`working_with_accelerators` (hardware profiles) guide** — every vendor
  accelerator page (NVIDIA/Gaudi/AMD) and the Kueue chapter all forward-
  reference "Working with hardware profiles" as the shared next step. Not
  ingested; noted as a gap on every page that references it.
- **IBM Spyre accelerators** — mentioned only in the original
  [[enabling-accelerators]] stub; this guide's Chapter 7 doesn't cover it
  either. Still an open gap.
- **Kueue CRD component pages** (`ResourceFlavor`, `ClusterQueue`,
  `LocalQueue`, the `Kueue` CR itself) — described richly across Ch8/Ch9 but
  not split into their own `component` pages (only [[datasciencecluster]]
  and [[dscinitialization]] exist as CR component pages today). Given the
  volume already ingested this round, treating these as a candidate for a
  future ingest pass rather than adding 4 more pages now — flagging instead
  of silently skipping.
- **`GatewayConfig` / Gateway API objects** — same call as Kueue CRDs:
  richly described procedurally (Ch10) but not split into a `component`
  page. Candidate for a future pass.
- Did not touch [[rhoai-dashboard-not-accessible]] despite
  [[troubleshooting-centralized-authentication-service]] covering a
  similarly-named 403 symptom — confirmed they're genuinely different
  failure modes (generic post-install dashboard-unreachable vs.
  OIDC-authenticated-but-unauthorized) and cross-linked them instead of
  merging.
- "Deploying models" / KServe serving-runtime details referenced from
  [[creating-project-scoped-resources]] — out of scope for this guide, not
  ingested.
- OpenShift-side docs referenced throughout (OADP backup/restore, audit-log
  policy config, SR-IOV Operator install, machine-set scaling, etc.) are
  left as named-but-not-ingested per existing wiki convention.

### Findings

- Two whole sections of Chapter 13 (13.1 Operator logger, 13.2 audit
  records) are word-for-word duplicates of already-ingested install-guide
  content — the guide's own text literally repeats itself across manuals.
  Worth knowing for future ingests: check overlap at the *section* level,
  not just the *page* level, since RHOAI's docs set reuses content across
  guides.
- Kueue's `managementState` story changed meaningfully since the install
  guide was ingested: **as of 2.24**, embedded Kueue (`Managed`) is
  deprecated in favor of the Red Hat build of Kueue Operator (`Unmanaged`).
  The existing [[installing-distributed-workloads-components]] page still
  correctly showed `Unmanaged` as the only supported value, so no
  contradiction to record on a version page — but
  [[migrating-to-red-hat-build-of-kueue-operator]] documents the
  transition for anyone still on the deprecated path.

## [2026-08-25] ingest | 29 unit(s)

- Managing OpenShift AI > Chapter 1. Managing users and groupsCopy linkLink copied to clipboard! > 1.1. Overview of user types and permissionsCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 1. Managing users and groupsCopy linkLink copied to clipboard! > 1.3. Adding users to OpenShift AI user groupsCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 1. Managing users and groupsCopy linkLink copied to clipboard! > 1.5. Deleting usersCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 2. Creating custom workbench imagesCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 2. Creating custom workbench imagesCopy linkLink copied to clipboard! > 2.2. Creating a custom image from your own imageCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 2. Creating custom workbench imagesCopy linkLink copied to clipboard! > 2.3. Enabling custom images in OpenShift AICopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 3. Managing applications that show in the dashboardCopy linkLink copied to clipboard! > 3.1. Adding an application to the dashboardCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 3. Managing applications that show in the dashboardCopy linkLink copied to clipboard! > 3.4. Showing or hiding information about available applicationsCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 6. Customizing component deployment resourcesCopy linkLink copied to clipboard! > 6.1. Overview of component resource customizationCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 7. Enabling acceleratorsCopy linkLink copied to clipboard! > 7.1. Enabling NVIDIA GPUsCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 7. Enabling acceleratorsCopy linkLink copied to clipboard! > 7.2. Intel Gaudi AI Accelerator integrationCopy linkLink copied to clipboard!
- Managing OpenShift AI > Chapter 7. Enabling acceleratorsCopy linkLink copied to clipboard! > 7.3. AMD GPU IntegrationCopy linkLink copied to clipboard!

## [2026-08-25] ingest | raw/rhoai/3.5/configuring-your-model-serving-platform.md

**Source:** `raw/rhoai/3.5/configuring-your-model-serving-platform.md` (9 units, ~12K tokens — About
model-serving platforms, ServingRuntime/InferenceService CRDs, accelerator runtimes, configuring
model servers incl. speculative decoding/multi-modal, adding custom and tested-and-verified
runtimes, NVIDIA NIM platform, customizing model deployments, legal notice).

**Created (10 pages, all `rhoai-3.5`):**
- concept: [[model-serving-platforms-overview]], [[accelerator-serving-runtimes]]
- component: [[kserve-model-serving-crds]] (covers both `ServingRuntime` and `InferenceService`
  CRDs on one page — they're introduced together in the source and are tightly coupled; splitting
  further felt like over-splitting a 9-unit guide)
- procedure: [[enabling-model-serving-platform]],
  [[configuring-speculative-decoding-and-multimodal-inferencing]],
  [[adding-custom-model-serving-runtime]], [[adding-tested-and-verified-runtime]],
  [[enabling-nvidia-nim-model-serving-platform]],
  [[customizing-model-serving-runtime-parameters]] (merged source subsections 4.1 "Customizing the
  parameters of a deployed model-serving runtime", 4.2 "Customizable model serving runtime
  parameters" [a small reference table], and 4.3 "Customizing the vLLM model-serving runtime" [the
  Llama/Granite/Mistral worked examples] into one page — 4.2 and 4.3 are both extensions of the same
  procedure, not separate things), [[setting-default-model-deployment-strategy]].

**Updated:** none. Did not have a verified dump of existing page content this run (only
`index.md`'s one-line summaries), so did not attempt to rewrite any existing page's full content.

**Deliberately not touched / flagged uncertainty:**
- [[installing-rhoai-components]] and [[configuring-kserve-oauth-proxy-sidecar-resources]] are the
  two pre-existing pages closest to this guide's territory (DSC `kserve` component; KServe OAuth
  proxy sidecar sizing). New pages here link *out* to both by slug. Could **not** verify whether
  either page already covers ServingRuntime/InferenceService CRDs, accelerator runtime mapping, or
  the NIM platform, since only the index summary was available, not their content — treating the
  new pages as additive and non-duplicating on the assumption that no dedicated KServe/model-serving
  pages existed yet, but this is unverified. Recommend a follow-up pass reading those two pages'
  actual content to (a) confirm no duplication and (b) add reverse
  [[kserve-model-serving-crds]]/[[model-serving-platforms-overview]] cross-links from their side.
- Did not create pages for: "Working with hardware profiles" (referenced repeatedly for Gaudi/AMD/
  Spyre but documented in a different guide, out of scope here); the Node Feature Discovery
  Operator (OCP-side operator, referenced only as a prerequisite link, not detailed in this guide);
  Red Hat's "Supported Configurations for 3.x" and "Tested and verified runtimes" articles (external,
  not raw/ content — summarized instead of duplicated, per the moving-value rule).
- Legal Notice unit: administrative boilerplate, skipped entirely.
- Kept accelerator image tags/SHA digests and exact resource sizings out of version-agnostic prose
  per rule 2 (moving values).

**Gaps / findings:**
- The guide repeatedly forward-references [[enabling-nvidia-gpus]], [[enabling-intel-gaudi-accelerators]],
  and [[enabling-amd-gpus]] (all already indexed) as prerequisites — linked from
  [[accelerator-serving-runtimes]] and the relevant procedures.
- [[enabling-nvidia-nim-model-serving-platform]] is new ground — no NIM-specific page existed
  before this ingest.
- `sources` line ranges verified directly against
  `raw/rhoai/3.5/configuring-your-model-serving-platform.md` via `grep -n` of its heading lines.

## [2026-08-25] ingest | 9 unit(s)

- Configuring your model-serving platform > Chapter 1. About model-serving platformsCopy linkLink copied to clipboard! > 1.1. About model servingCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 1. About model-serving platformsCopy linkLink copied to clipboard! > 1.2. Model-serving runtimesCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 1. About model-serving platformsCopy linkLink copied to clipboard! > 1.3. Model-serving runtimes for acceleratorsCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 2. Configuring model serversCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 2. Configuring model serversCopy linkLink copied to clipboard! > 2.3. Adding a custom model-serving runtimeCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 2. Configuring model serversCopy linkLink copied to clipboard! > 2.4. Adding a tested and verified runtimeCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 3. Configuring model servers on the NVIDIA NIM model serving platformCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Chapter 4. Customizing model deploymentsCopy linkLink copied to clipboard! > 4.2. Customizable model serving runtime parametersCopy linkLink copied to clipboard!
- Configuring your model-serving platform > Legal NoticeCopy linkLink copied to clipboard!

## [2026-08-26] ingest | Kuadrant core building blocks (auth, rate-limiting, token-rate-limiting) — inference-gateway slice 1

Ingested the first Kuadrant slice: the Gateway API-based policy engine RHOAI's MaaS (Models-as-a-Service) will build an "inference gateway" on top of. This slice covers only the CORE building blocks — the `Kuadrant` CR, `AuthPolicy`, `RateLimitPolicy`, `TokenRateLimitPolicy`, their three concept overviews, and one full walkthrough plus one token-rate-limiting tutorial. Scoped explicitly to `products: [kuadrant]` (upstream, no Red Hat product version) — this is genuinely new territory in the wiki, no prior Kuadrant/Gateway API pages existed.

**New pages (9):**
- component: [[kuadrant-cr]], [[kuadrant-authpolicy]], [[kuadrant-ratelimitpolicy]], [[kuadrant-tokenratelimitpolicy]]
- concept: [[kuadrant-auth-overview]], [[kuadrant-rate-limiting-overview]], [[kuadrant-token-rate-limiting-overview]]
- procedure: [[kuadrant-secure-protect-connect-walkthrough]], [[kuadrant-authenticated-token-rate-limiting-tutorial]]

**Updated pages:** none — no existing Kuadrant/Gateway API pages to update.

**Design choices:**
- Split each of the 4 CRDs (Kuadrant CR, AuthPolicy, RateLimitPolicy, TokenRateLimitPolicy) into its own `component` page rather than one combined page, since each is independently referenced/configured and has substantial field-level detail.
- Made the 3 overview docs (`auth.md`, `rate-limiting.md`, `token-rate-limiting.md`) separate `concept` pages preceding the component pages, per the ingest brief's instruction — they motivate *why* before the component pages get into field detail.
- Did not create a standalone page for the 9 toystore example manifests; folded their concrete patterns into the relevant component/procedure pages as worked examples with citations to the actual `examples/toystore/*.yaml.md` files (e.g. the two-tier AuthPolicy example on [[kuadrant-authpolicy]], the four-limit RateLimitPolicy examples on [[kuadrant-ratelimitpolicy]], the free-tier TokenRateLimitPolicy example on [[kuadrant-tokenratelimitpolicy]]).
- The full walkthrough exercises `TLSPolicy`/`DNSPolicy` steps that are out of scope for this ingest (DNS/TLS/mTLS-x509 are a follow-up ingest's job); documented those steps procedurally (what they do, that they exist) without creating component pages for them, and flagged them explicitly as out-of-scope forward references.
- Cross-linked outward to existing RHOAI pages [[model-serving-platforms-overview]] and [[kserve-model-serving-crds]] wherever TokenRateLimitPolicy's kServe/vLLM backend compatibility is documented — this is the concrete mechanism a later MaaS/RHOAI ingest would build a token-cost-billing page on top of, but no such integration page exists yet and none was fabricated here.
- Explicitly left out of scope per the brief: DNS, TLS, mTLS/x509 client-cert auth, OIDC federation, observability/tracing, CEL extensions, and the extension SDK — each is only noted as an external forward reference where the source material mentions it in passing (e.g. the `Kuadrant` CR's `mtls`/`observability` spec fields are documented as fields that exist, without expanding into full guides).

**What I chose not to change:** the existing ~140 rhoai-3.5 pages (components, concepts, procedures, errors) — none overlap with this ingest; no updates were needed there. No error-type pages were created — none of the ingested source material described a symptom/diagnostic/fix pattern.

Sources: raw/kuadrant-operator/doc/reference/{kuadrant,authpolicy,ratelimitpolicy,tokenratelimitpolicy}.md; raw/kuadrant-operator/doc/overviews/{auth,rate-limiting,token-rate-limiting}.md; raw/kuadrant-operator/doc/user-guides/full-walkthrough/secure-protect-connect.md; raw/kuadrant-operator/doc/user-guides/tokenratelimitpolicy/authenticated-token-ratelimiting-tutorial.md; raw/kuadrant-operator/examples/toystore/*.yaml.md (9 files).

## [2026-08-26] ingest | 31 unit(s)

- Kuadrant Auth > Secure API access with authentication and authorization > Common authentication and authorization jobs
- Kuadrant Auth > Choosing the right authentication method > Choosing an authorization method
- Kuadrant Auth > Apply consistent security policies across your API infrastructure > Establish platform-wide security baselines with Gateway-targeted policies
- Kuadrant Auth > Apply consistent security policies across your API infrastructure > Apply policies conditionally based on request attributes with `when` conditions
- Kuadrant Rate Limiting > How it works > Envoy's Rate Limit Service Protocol
- Kuadrant Rate Limiting > Using the RateLimitPolicy > Targeting a Gateway networking resource
- Kuadrant Rate Limiting > Implementation details
- Kuadrant Token Rate Limiting > How it works > Token Tracking Protocol
- Kuadrant Token Rate Limiting > Integration with AuthPolicy
- The AuthPolicy Custom Resource Definition (CRD) > AuthPolicySpec
- The AuthPolicy Custom Resource Definition (CRD) > AuthPolicyCommonSpec > AuthScheme
- The AuthPolicy Custom Resource Definition (CRD) > AuthPolicyCommonSpec > NamedPattern
