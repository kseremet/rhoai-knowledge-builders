---
type: concept
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/installing-and-uninstalling-openshift-ai-self-managed.md#L60-79
verified: 2026-08-25
confidence: documented
---

# RHOAI operator update channels

The [[rhoai-operator]]'s OLM `Subscription` specifies an update channel,
which controls which minor version the Operator tracks and how much support
and release cadence that tracking gets. Channel choice is set at
subscription time (see [[installing-the-rhoai-operator]]) and can be changed
later to move to a different channel.

## Channels, as of 3.5

| Channel | Support window | Release frequency | Use for |
|---|---|---|---|
| `fast` / `fast-x.y` | 1 month full support | monthly | production wanting latest features, automatic updates |
| `stable` | 3 months full support | every 3 months | production prioritizing stability, automatic updates |
| `stable-x.y` | 7 months full support | every 3 months | production that wants to plan/control the move to the next stable release |
| `eus-x.y` | 7 months full support + 11 months Extended Update Support | every 9 months | enterprise environments that can't upgrade within a 7-month window |
| `alpha` | 1 month full support | monthly | development/early-access, not covered by production SLAs |

Notes specific to 3.5:

- RHOAI 3.0 was available only through `fast-3.x`, not the general `fast`
  channel.
- `embedded` and `beta` are legacy channels being removed in a future
  release — do not select them for a new install.

## Upgrade behavior

By default the Operator follows a **sequential update process**: if several
versions separate the current and target version, OLM upgrades through each
intermediate version before reaching the target.

- **Automatic** update approval: OLM auto-upgrades through the chain to the
  *latest* available version in the channel.
- **Manual** update approval: a cluster administrator must approve each
  sequential step.

For release cadence and lifecycle detail per channel, see the [Red Hat
OpenShift AI Self-Managed Life Cycle](https://access.redhat.com/support/policy/updates/rhoai-sm/lifecycle)
Knowledgebase article (external, not ingested).

## Where this is set

Channel is a `Subscription` field (`spec.channel`) created in
[[installing-the-rhoai-operator]], and update-approval mode (`Automatic` /
`Manual`) is set at the same time via the web console install flow.
