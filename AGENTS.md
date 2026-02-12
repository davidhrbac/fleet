# AGENTS.md

This repository contains Rancher Fleet bundles and Kubernetes manifests.
Use this file as the default guidance for agentic changes in this repo.

## Scope and layout

- `kured/` deploys the Kured Helm chart with a dev overlay.
- `rancher-inventory/` deploys Rancher inventory resources and a client Deployment.
- `rancher-monitoring/` deploys Rancher Monitoring CRDs and chart via Fleet.

There is no application source code in this repo.
Changes are primarily YAML edits to Fleet bundles and Kubernetes manifests.

## Build, lint, test commands

No build, lint, or test scripts are defined in this repository.
There are no unit tests or single-test runners here.

Use the following optional validation commands when available in your toolchain:

- Validate YAML syntax (if `yamllint` is installed):
  `yamllint .`
- Dry-run manifests (if `kubectl` is installed):
  `kubectl apply --dry-run=client -f rancher-inventory/manifests`
- Render the Kustomize overlay (if `kustomize` is installed):
  `kustomize build kured/overlays/dev`
- Validate Fleet Helm bundles with Helm (if `helm` is installed):
  `helm repo add rancher https://charts.rancher.io`
  `helm repo add kured https://kubereboot.github.io/charts`
  `helm template rancher-monitoring rancher/rancher-monitoring -n cattle-monitoring-system`
  `helm template rancher-monitoring-crd rancher/rancher-monitoring-crd -n cattle-monitoring-system`
  `helm template kured kured/kured -n kured`

If a command is not available, document that validation was skipped.

## Single-test guidance

- There are no unit tests or single-test runners in this repo.
- Validate only the bundle you changed (Helm template, kustomize build, or
  kubectl dry-run) to keep checks focused.
- When unsure, prioritize dry-running the specific manifests you edited.

## Cursor or Copilot rules

No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` files
were found in this repository. No extra editor rules apply.

## YAML style and formatting

- Use 2-space indentation; never use tabs.
- Keep keys in lowercase unless Kubernetes schema requires otherwise.
- Keep list items aligned; avoid trailing whitespace.
- Prefer double quotes only when a string contains special characters, colons,
  or leading/trailing spaces. Otherwise use plain scalars.
- Keep file ordering stable; do not reorder keys unless required by a change.
- Preserve existing comments and blank lines that aid readability.
- Avoid anchors and aliases unless there is a clear reuse need.
- Keep boolean values as `true`/`false`, not quoted strings.
- Keep numeric values as numbers unless the schema requires strings.
- Use single-document YAML files; do not add `---` unless the file already has it.
- Keep line lengths reasonable; wrap long scalars when they hurt readability.

## Code style guidelines (general)

- This repo does not contain application code, imports, or types.
- Follow Kubernetes schema conventions rather than language-specific rules.
- Prefer declarative intent; avoid imperative hooks or scripts in manifests.
- Keep patches minimal: only include fields that differ from the base.
- Prefer explicit values over defaults only when behavior must change.
- Avoid commented-out configuration unless it documents a required workflow.

## Kubernetes manifest conventions

- Always include `apiVersion`, `kind`, and `metadata.name`.
- Set `metadata.namespace` for namespaced resources.
- Keep `metadata.labels` stable; prefer `app` or `app.kubernetes.io/*` when
  adding new labels.
- Use `resources` requests/limits for containers; prefer explicit CPU/memory.
- Prefer `restartPolicy: Always` for Deployments unless the workload requires
  a different policy.
- For workloads, consider `livenessProbe` and `readinessProbe` if the change
  introduces long-running containers that need health checks.
- Avoid hardcoding secrets; use `valueFrom.secretKeyRef` for sensitive data.
- Keep service account and RBAC changes scoped to the minimum privileges.
- Keep selectors in sync with pod template labels.
- Avoid changing `metadata.name` of existing resources unless required.
- Set `imagePullPolicy` only when necessary; let the cluster default otherwise.
- Prefer `envFrom` for ConfigMaps when multiple keys are used consistently.
- Use `securityContext` and `podSecurityContext` when privilege changes are needed.
- Use `terminationGracePeriodSeconds` when shutdown behavior matters.

## Fleet bundle conventions

- Fleet bundles are configured via `fleet.yaml` with `defaultNamespace` and
  an optional `helm` section.
- Prefer explicit `helm.releaseName` values and keep them stable.
- Use `helm.values` only for settings that must differ from the chart defaults.
- Keep chart versions pinned when a version is specified by convention.
- Avoid adding inline secrets in `helm.values`; use Kubernetes Secrets or
  external secret management instead.
- Keep `fleet.yaml` focused on bundle-level configuration; avoid unrelated manifests.
- When enabling Helm values, mirror the chart structure and avoid typos.
- Avoid enabling features that require CRDs without ensuring the CRDs exist.
- For per-cluster differences, prefer `targetCustomizations` with cluster labels
  and patches rather than forking base manifests.

## Kustomize overlay conventions

- Keep overlays in `kured/overlays/<env>` and base resources untouched.
- Prefer `patchesStrategicMerge` with minimal diffs focused on required fields.
- Do not duplicate base values in overlays unless they must change.
- Use overlay-specific `values.yaml` only when the chart consumes it.
- Keep patch files scoped to one resource when possible.
- Avoid replacing entire lists unless all entries must change.
- Ensure patches match resource `kind`, `metadata.name`, and `metadata.namespace`.

## Naming conventions

- Resource names should be DNS-1123 compliant and lowercase.
- Use hyphenated names (`rancher-inventory-client`, `readonly-role`).
- Keep names consistent across `metadata.name`, labels, and selectors.
- For namespaces, use a single, purpose-specific name (no environment suffix
  unless required by the cluster standard).
- Use clear label values that match the resource purpose.
- Keep `releaseName` aligned with chart and bundle names.

## Error handling and safety

- Favor declarative changes that can be applied idempotently.
- Add or update RBAC with least privilege, and document why in the PR.
- When updating image tags, verify the image exists and is compatible.
- Avoid breaking changes in shared namespaces without coordination.
- If a manifest change could cause downtime, note it in the summary.
- When changing probes or resource limits, consider rollout impact.
- Avoid deleting resources unless the intent is explicit and documented.
- Ensure namespaces exist before adding namespaced resources in a new bundle.

## Change scope

- Keep changes limited to the bundle or manifest you were asked to modify.
- Do not reformat unrelated YAML or reorder keys without a functional reason.
- Avoid widening RBAC or adding cluster-wide permissions unless required.

## Review checklist for changes

- YAML is valid and formatted consistently.
- Namespaces and selectors are correct and match labels.
- Secrets are referenced via `secretKeyRef`, not in plaintext.
- Helm values changes are scoped to required settings.
- Kustomize overlay patches are minimal and targeted.
- No accidental changes to unrelated bundles.
