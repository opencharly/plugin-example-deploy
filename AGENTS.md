# AGENTS.md — plugin-example-deploy

Standalone plugin repo for the `exampledeploy` capability
(`deploy:exampledeploy`) — the reference external deploy target and the F2
executable-deploy-channel witness. The plugin is a Go module at
`candy/plugin-example-deploy/` (module path
`github.com/opencharly/plugin-example-deploy/candy/plugin-example-deploy`); the
root `charly.yml` only declares `discover: candy` (plus the R10 bed) so the repo
is a project and its candy is scanned.

Canonical files:

- `candy/plugin-example-deploy/charly.yml` — the `plugin-example-deploy:` candy
  entity (`plugin:` block, `plan:` steps).
- `candy/plugin-example-deploy/plugin.go` — the provider (`NewProvider()` +
  `NewMeta()`) and the deploy walk (`applyStep`, `PutFile`, `RunHostStep`).
- `candy/plugin-example-deploy/schema/exampledeploy.cue` — the self-contained
  `#ExampledeployInput`.
- `charly.yml` — the root manifest + the `check-exampledeploy:` R10 bed.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-internals:plugin` — the plugin authoring reference: the `plugin:`
  block, the unified Provider model (incl. the `deploy` class), the per-plugin
  CUE-schema contract, placement. Load before touching the provider or schema.
- `/charly-internals:install-plan` — the `InstallPlan` IR, the `OpExecute`
  reverse channel, and the `pluginDeployTarget` lifecycle.
- `/charly-check:check` — the R10 disposable bed surface the
  `check-exampledeploy:` entity is authored through.
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `go build ./...` in `candy/plugin-example-deploy/` — compile the plugin module.
- `go test ./...` in `candy/plugin-example-deploy/` — the plugin's Go tests.
- `charly box validate` at the repo root — the structural check (the candy +
  `plugin:` block, CUE schema, and the `check-exampledeploy:` bed).
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate.
- R10: the `check-exampledeploy:` disposable bed is the sole proof of the
  external deploy lifecycle (Add/Test/Update/Del) over the reverse channel.

## Modify this repo

- Edit the `plugin-example-deploy:` candy entity, the Go source, and
  `schema/exampledeploy.cue` **together**.
- The plugin is **out-of-process** (host-built and served over go-plugin gRPC);
  keep every scratch path under `/tmp` so teardown leaves zero residue.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
