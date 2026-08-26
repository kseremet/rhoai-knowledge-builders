---
type: concept
products: [rhoai, ocp]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L1643-1674
verified: 2026-08-25
confidence: documented
---

# Centralized authentication service

**Why this exists**: on Red Hat OpenShift Service on AWS (ROSA) and
on-prem OCP 4.20+, the built-in OpenShift OAuth server has limits on direct
OpenID Connect (OIDC) configuration — the internal `oauth` service is
disabled, breaking applications that depend on `oauth-proxy` sidecars.
RHOAI's centralized authentication service configures an external OIDC
provider directly, routing ingress for all RHOAI services behind a single
domain (a Gateway), instead of per-service OAuth proxies.

## What it provides

- **Centralized authentication**: one authentication service, one client
  ID/secret from the external OIDC provider.
- **Simplified backend services**: backends can assume all incoming
  traffic is already authenticated and carries the necessary user headers.
- **Authorization still per-service**: services still handle authorization
  at the service/pod level (e.g. via a `kube-rbac-proxy` sidecar) — the
  Gateway only centralizes *authentication*.
- **Encrypted traffic**: Gateway-to-backend traffic is fully TLS-encrypted.

Based on Gateway API support in OpenShift (OCP) 4.19.9+.

## Authentication methods

- **User authentication via OIDC** — interactive browser access. The
  Gateway redirects unauthenticated requests to the configured external
  OIDC provider for login. See
  [[configuring-oidc-centralized-authentication]].
- **Service account token authentication** — programmatic/CLI/automated
  access. The Gateway validates Kubernetes service account bearer tokens
  against the cluster API server. See
  [[configuring-service-account-token-authentication]].

## See also

[[configuring-custom-ca-for-oidc]] for trusting a self-signed OIDC
provider certificate, and
[[troubleshooting-centralized-authentication-service]] for common
Gateway/OIDC failure symptoms.
