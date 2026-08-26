---
type: procedure
products: [rhoai, ocp]
versions: [rhoai-3.5, ocp-4.20]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed-in-a-d.md#L225-269
verified: 2026-08-25
confidence: documented
---

# Mirroring images to a private registry for a disconnected installation

Wholly disconnected-environment-specific: mirrors the container images the
[[rhoai-operator]] needs to a private registry, so [[installing-the-rhoai-operator]]
can proceed on a cluster with no direct route to Red Hat's remote registries.
This is step 2 of the disconnected high-level install sequence (requirements
→ **mirror images** → operator → components → ...), inserted between
"confirm requirements" and "install the Operator" that the connected-cluster
install flow does not have at all.

Uses the *mirror registry for Red Hat OpenShift* (a small-scale registry) as
an optional target — any container registry already available in the
disconnected environment works too.

## Preconditions

- Cluster administrator access to a running OpenShift cluster, version 4.19
  or later.
- Credentials for [Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/).
- A mirroring machine running Linux, with 100 GB free space and internet
  access (to `cdn.redhat.com`, `subscription.rhn.redhat.com`,
  `registry.access.redhat.com`, `registry.redhat.io`, `quay.io` — see the
  disconnected note on [[installing-the-rhoai-operator]] for the full
  domain list).
- OpenShift CLI (`oc`) installed on the mirroring machine.
- Component requirements reviewed so any additional operators are included
  in the mirror set, not just the RHOAI Operator itself.

**Note**: this procedure uses **oc-mirror plugin v2** — v1 is deprecated.

## Steps

1. Create a mirror registry (see OpenShift's "Creating a mirror registry
   with mirror registry for Red Hat OpenShift" — external, not ingested), or
   identify an existing target registry.
2. Install the `oc-mirror` OpenShift CLI plugin **v2** on the mirroring
   machine (external OCP doc, not ingested). The v1 plugin is deprecated;
   use v2 for continued support.
3. Create a container image registry credentials file that allows mirroring
   images from Red Hat to your mirror (external OCP doc, not ingested).
4. Open the example image set configuration file (`rhoai-<version>.md`) from
   the [disconnected installer helper](https://github.com/red-hat-data-services/rhoai-disconnected-install-helper/tree/main)
   repository. It lists required **Additional images** plus older
   **Unsupported images** kept only for reference/compatibility.
5. Using that example, create `imageset-config.yaml` for your deployment.
   To list available OpenShift versions: `oc-mirror list operators`. To list
   channels for a package at a specific OpenShift version:
   `oc-mirror list operators --catalog=registry.redhat.io/redhat/redhat-operator-index:v4.19 --package=<package_name>`.
   Example:
   ```yaml
   kind: ImageSetConfiguration
   apiVersion: mirror.openshift.io/v1alpha2
   mirror:
     operators:
       - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.19
         packages:
           - name: rhods-operator
             channels:
               - name: stable
                 minVersion: 2.25.0
                 maxVersion: 2.25.0
           - name: <additional_operator_name>
             channels:
               - name: stable
         additionalImages:
           - name: <additional_image_name>
   ```
   The example configurations are for demonstration only; consult OCP's
   "Image set configuration parameters" and "Image set configuration
   examples" docs (not ingested) to tune for your deployment.
6. Download the image set to the mirroring machine (target directory must
   start with `file://`):
   `$ oc mirror -c imageset-config.yaml file://<mirror_rhoai> --v2`
   Tip: if you hit `tls: failed to verify certificate: x509: certificate
   signed by unknown authority` and want to ignore it, set `skipTLS: true`
   in the image set configuration and re-run.
7. Verify the `.tar` files were created: `$ ls <mirror_rhoai>` — expect
   output like `mirror_000001.tar, mirror_000002.tar` (an `archiveSize`
   setting in the config can split the set into multiple files).
8. Optional: verify total `.tar` size is around 75 GB:
   `$ du -h --max-depth=1 ./<mirror_rhoai>/` — if significantly smaller,
   re-run the `oc mirror` command from step 6.
9. Upload the image set to the target mirror registry:
   `$ oc mirror -c imageset-config.yaml --from file://<mirror_rhoai> docker://<registry.example.com:5000> --v2`
   Tip: for the same TLS-verification error here, run
   `$ oc mirror --dest-tls-verify false --from=./<mirror_rhoai> docker://<registry.example.com:5000> --v2`.
10. Log in to the target OpenShift cluster as a cluster administrator.
11. Verify the generated `ImageDigestMirrorSet` and `CatalogSource` YAML
    files exist: `$ ls <mirror_rhoai>/working-dir/cluster-resources/` —
    expect `cs-redhat-operator-index.yaml idms-oc-mirror.yaml`.
12. Install the generated resources into the cluster:
    `$ oc apply -f <oc_mirror_workspace_path>/working-dir/cluster-resources`

## Verification

- `$ oc get catalogsource,pod -n openshift-marketplace` — expect at least
  one catalog and two pods.
- Web console → **Operators** → **OperatorHub** → confirm the Red Hat
  OpenShift AI Operator tile is shown.
- If additional operators were mirrored, confirm they also appear in
  OperatorHub.

## Next step

[[installing-the-rhoai-operator]] — the mirrored `CatalogSource` this
procedure creates is named `cs-redhat-operator-index` by default (per the
verification step's file listing), which the disconnected install's
`Subscription` object references via `spec.source` instead of the
connected-cluster default of `redhat-operators` — see the disconnected note
on that page.

## Additional resources

- [Mirroring images for a disconnected installation by using the oc-mirror plugin v2](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/disconnected_environments/about-installing-oc-mirror-v2)
