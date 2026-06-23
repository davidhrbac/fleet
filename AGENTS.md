# AGENTS.md

## Repo Shape

- This repo is only Rancher Fleet/Kubernetes YAML; there is no application code, package manager, CI workflow, or test suite here.
- `kured/fleet.yaml` deploys the `kured` Helm chart from `https://kubereboot.github.io/charts` into namespace `kured`.
- `kured/overlays/dev/` is a loose dev overlay patch/values area, not a standalone renderable base; do not assume `kustomize build kured/overlays/dev` is a valid check.
- `rancher-monitoring/00-monitoring-crd/` must be deployed before `rancher-monitoring/01-monitoring/`; the second bundle installs the chart and assumes the CRDs exist.
- `rancher-inventory/fleet.yaml` has `defaultNamespace: cattle-global-data`, but its manifests create/use the `rancher-inventory` namespace; preserve that unless intentionally changing Fleet behavior.

## Validation

- No repo-local build/lint/test commands exist. Validate only the bundle you changed.
- YAML syntax, if available: `yamllint .`
- Rancher inventory manifests without requiring CRDs locally: `kubectl apply --dry-run=client --validate=false -f rancher-inventory/manifests`
- Helm render Kured: `helm template kured kured --repo https://kubereboot.github.io/charts -n kured`
- Helm render monitoring CRDs: `helm template rancher-monitoring-crd rancher-monitoring-crd --repo https://charts.rancher.io -n cattle-monitoring-system`
- Helm render monitoring chart: `helm template rancher-monitoring rancher-monitoring --repo https://charts.rancher.io -n cattle-monitoring-system`
- If a tool is missing or a dry-run needs cluster access, say validation was skipped and why.

## Rancher Inventory Gotchas

- The `inventory-proxy` Fleet customization applies only to clusters labeled `use-inventory-proxy: "true"`.
- Proxy settings are intentionally not stored in Git; labeled clusters must provide Secret `inventory-proxy-env` in namespace `rancher-inventory`.
- The client Deployment expects Secret `rancher-api-token` with key `token`; do not inline this token.
- `00_globalrole.yaml` is cluster-scoped Rancher RBAC and uses `fleet.cattle.io/skip-namespace-scoping: "true"`; keep that annotation when editing the GlobalRole.

## Edit Rules

- Keep changes scoped to the touched bundle and avoid reformatting unrelated YAML.
- Preserve existing Fleet `helm.releaseName` values and Rancher chart URLs unless the task is explicitly to change deployment identity/source.
- For per-cluster differences, prefer Fleet `targetCustomizations` with labels and overlays rather than forking whole bundles.
- Keep secrets out of `fleet.yaml`, Helm values, and manifests; reference Kubernetes Secrets instead.
