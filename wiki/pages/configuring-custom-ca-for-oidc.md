---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1767-1789
verified: 2026-08-25
confidence: documented
---

# Configuring a custom CA for OIDC

Makes the Gateway controller trust a self-signed (or otherwise
non-public-CA) OAuth/OIDC provider certificate, for a secure connection
between `kube-auth-proxy` and the provider configured in
[[configuring-oidc-centralized-authentication]].

## Preconditions

- Cluster administrator access.
- The CA certificate for your OAuth/OIDC provider.

## Steps

1. Create a Secret with the CA certificate (the one that signed the
   provider's TLS certificate) in `openshift-ingress`:
   ```
   oc create secret generic <oidc-ca-bundle> \
     --from-file=ca.crt=</path/to/your-ca.crt> \
     -n openshift-ingress
   ```
2. Reference it on the `GatewayConfig` resource's
   `spec.providerCASecretName`:
   ```yaml
   apiVersion: opendatahub.io/v1alpha1
   kind: GatewayConfig
   metadata:
     name: gatewayconfig
   spec:
     # ... other spec fields (subdomain, ingressMode, etc.) ...
     providerCASecretName: oidc-ca-bundle
     # Optional, dev/test only — do not use in production:
     # insecureSkipVerify: true
   ```
   The Gateway controller mounts the Secret and configures
   `kube-auth-proxy` to trust it.

## Verification

After saving `GatewayConfig`, the Operator reconciles and updates
`kube-auth-proxy`:

- `kube-auth-proxy` pods in `openshift-ingress` restart and reach
  `Running`.
- Logging in to the dashboard succeeds without TLS certificate errors.
