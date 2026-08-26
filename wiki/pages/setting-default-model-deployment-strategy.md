---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources: [raw/rhoai/3.5/configuring-your-model-serving-platform.md#L578-602]
verified: 2026-08-25
confidence: documented
---

# Setting a default cluster-wide model deployment strategy

Sets the default deployment strategy (**Rolling update** or **Recreate**) applied to new model
deployments across the cluster — distinct from
[[customizing-model-serving-runtime-parameters]], which is per-deployment.

## Preconditions

- Logged in to the RHOAI dashboard as an OpenShift AI administrator.
- Model serving is enabled on the cluster — see [[enabling-model-serving-platform]].

## Steps

1. **Settings → Cluster settings** in the dashboard.
2. **General settings** tab.
3. Scroll to **Model deployment options**.
4. In **Default deployment strategy**, choose **Rolling update** or **Recreate**.
5. Click **Save changes**.

## Verification

Deploy a new model ([[model-serving-platforms-overview]] flow); on the **Advanced settings** page
of the deployment wizard, the **Deployment strategy** section's preselected value matches the new
default.

## Rollback

Repeat the steps and select the previous strategy.

## See also

- [[customizing-model-serving-runtime-parameters]]
- [[enabling-model-serving-platform]]
