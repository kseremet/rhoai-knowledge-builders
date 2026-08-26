---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L642-692
verified: 2026-08-25
confidence: documented
---

# How RHOAI handles certificates

The concept a reader needs before any of the certificate procedures make
sense: RHOAI's default CA-bundle mechanism, and the three
management-state options that change or disable it.

## The mechanism

When [[rhoai-operator]] is installed, it creates an empty
`odh-trusted-ca-bundle` ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app.kubernetes.io/part-of: opendatahub-operator
    config.openshift.io/inject-trusted-cabundle: 'true'
  name: odh-trusted-ca-bundle
```

The label `config.openshift.io/inject-trusted-cabundle: 'true'` is an
OpenShift Cluster Network Operator (CNO) convention: CNO watches for it and
injects the cluster-wide CA bundle into the ConfigMap's `ca-bundle.crt`
key:

```yaml
data:
  ca-bundle.crt: |
    <BUNDLE OF CLUSTER-WIDE CERTIFICATES>
```

If a custom bundle is also configured (see
[[adding-certificates-to-custom-ca-bundle]]), the Operator adds a second
key, `odh-ca-bundle.crt`, alongside `ca-bundle.crt`.

This ConfigMap is what gives workbenches, model servers, and pipeline pods
their trusted-certificate view — see
[[configuring-certificate-for-workbenches]], [[configuring-certificate-for-pipelines]],
and [[using-cluster-wide-certificate-for-model-serving]] for how each
component consumes it, and [[ogx-server]] for how the RAG/OGX server
consumes it via its own CR field.

## The three management states

Set via `spec.trustedCABundle.managementState` on the [[dscinitialization]]
(DSCI) CR:

- **`Managed`** (default) — the Operator maintains `odh-trusted-ca-bundle`
  in every non-reserved namespace (new and existing), and keeps it in sync
  with `customCABundle`. Excluded: reserved/system namespaces like
  `default`, `openshift-*`, `kube-*`.
- **`Unmanaged`** — an administrator manages the ConfigMap by hand (e.g.
  via Ansible). Existing ConfigMaps are left in place when switching from
  `Managed`, but stop being auto-updated on `customCABundle` changes. Use
  this when the org already has its own trusted-CA-bundle tooling and
  doesn't want the Operator overwriting it. See
  [[managing-certificates-without-the-rhoai-operator]].
- **`Removed`** — the Operator deletes the ConfigMap everywhere and stops
  creating it in new namespaces. **This is the default after upgrading from
  RHOAI 2.7 or earlier to 3.5.** Chosen for reduced complexity / security
  posture (only approved CAs trusted, blocks unauthorized cert changes).
  See [[removing-the-ca-bundle]].

## The two ways to add a certificate

If an OpenShift AI component uses a self-signed certificate not already in
the cluster-wide CA bundle:

- Add it to the **cluster-wide** CA bundle — trusted everywhere
  automatically, simplest, but affects the whole cluster. See
  [[adding-certificates-to-cluster-wide-ca-bundle]].
- Add it to a **custom** CA bundle, separate from the cluster-wide one —
  scoped to just the services that need it, isolated from global config.
  See [[adding-certificates-to-custom-ca-bundle]].

Several components additionally support taking a CA bundle that applies
only to that component (pipelines: [[configuring-certificate-for-pipelines]];
OGX: [[configuring-ca-bundle-for-ogx-server]]).
