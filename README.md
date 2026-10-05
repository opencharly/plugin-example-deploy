# plugin-example-deploy

The reference **external deploy** plugin (`deploy:exampledeploy`) — proof that an
external plugin receives a real multi-step `InstallPlan`, executes its steps on
the venue, and pushes files back through the reverse channel.

charly's loader host-builds this plugin's provider binary and serves it
out-of-process. The host's plugin-side deploy target then invokes it
(`OpExecute`) with the deployment's `InstallPlan` views plus a venue descriptor,
and the plugin dials back through the SDK executor to:

- write apply/probe markers;
- walk and execute the plan's steps on the venue (a file-write step and an env
  shell-hook step);
- push a file via the reverse-channel `PutFile` leg;
- drive the `RunHostStep` host-engine leg for `BuilderStep` / `LocalPkgInstallStep`
  (the two kinds needing the host build engine).

It returns plugin-script reverse ops that the host records in the install ledger
and replays at `charly deploy del`, so teardown leaves zero residue.

## What it provides

| Capability | Surface |
|---|---|
| `deploy:exampledeploy` | the external deploy target — `OpExecute` over the E3b reverse channel |

The repo also declares the `check-exampledeploy:` R10 witness bed (a
`disposable: true` deploy) that drives the full Add/Test/Update/Del lifecycle
against disposable `/tmp` scratch dirs.

## How to use it

Compose the plugin candy as a deploy substrate:

```yaml
- '@github.com/opencharly/plugin-example-deploy/candy/plugin-example-deploy:<tag>'
```

## Layout

- `candy/plugin-example-deploy/` — the plugin module: `plugin.go` (the provider
  + `NewProvider()`/`NewMeta()` + the step walk, `PutFile`, and `RunHostStep`),
  `schema/exampledeploy.cue` (the self-contained `#ExampledeployInput`),
  `cmd/serve/main.go`.
- `charly.yml` — the root project manifest (`discover: candy`) plus the
  `check-exampledeploy:` R10 bed.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.

## Related

- Owning skill: `/charly-internals:install-plan` — the `InstallPlan` IR and the
  `pluginDeployTarget` lifecycle. This candy carries no `skill:` entity of its
  own; the gap is tracked in
  [opencharly/opencharly#291](https://github.com/opencharly/opencharly/issues/291).
- `/charly-internals:plugin` — the plugin/provider model.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI.
