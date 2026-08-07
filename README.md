# krateo-oas

OpenAPI → Kubernetes-kind GitOps publish path for the Krateo **API Builder** (KOG): a merged PR to this repo becomes a live cluster kind.

## What is this

This repo is **also a Helm chart**: its root is a singleton "API registry" chart that the
Krateo composition-dynamic-controller (CDC) renders. Merged API-Builder artifacts under
`apis/**` (RestDefinitions) and `configmaps/**` (their OpenAPI documents) become live
cluster resources; oasgen-provider then generates a CRD + controller per RestDefinition.
The human PR merge is the **hard gate**: nothing reaches the cluster before it.
Full picture: [docs/index.md](docs/index.md).

## Install

You don't install this repo by hand — CDC renders it from a pinned chart version via a
`CompositionDefinition` (see [docs/usage.md](docs/usage.md) for the exact CR and the
installer wiring). To render it locally (no cluster):

```sh
git clone https://github.com/krateo-platformops/oas.git
cp -R oas /tmp/krateo-oas-render
sed -i '' 's/CHART_VERSION/0.0.0-local/g' /tmp/krateo-oas-render/Chart.yaml   # Linux: sed -i
helm template krateo-oas /tmp/krateo-oas-render
```

## Configure

See [docs/configuration.md](docs/configuration.md). This is a singleton registry chart
with **no per-instance configuration** — its real content is the merged files:

| Setting | Default | Effect |
|---|---|---|
| `global` | `{}` (empty) | Helm globals passthrough; exists only so CDC generates a (trivial) CRD from `values.schema.json` |
| `apis/<kind>/restdefinition.yaml` | — | a merged RestDefinition = a generated Kubernetes kind |
| `configmaps/<kind>-oas.yaml` | — | the OpenAPI ConfigMap the RestDefinition's `oasPath` references |

## Examples

- [examples/echo-api](examples/echo-api) — a complete, valid API-Builder contribution (RestDefinition + OAS ConfigMap) you can validate and render locally, then propose by PR.

## Docs

- [docs/index.md](docs/index.md) — the map
- [docs/overview.md](docs/overview.md) — the GitOps publish path and why it's a chart, not a poller
- [docs/usage.md](docs/usage.md) — how CDC/the installer consume it; how to add or remove an API
- [docs/configuration.md](docs/configuration.md) — the (deliberately empty) values surface
- [docs/api.md](docs/api.md) — the file contract for contributions + what the chart emits
- [docs/examples.md](docs/examples.md) — examples index
- [docs/release.md](docs/release.md) — how a release ships (tag → OCI chart → pin bump)
- [docs/log.md](docs/log.md) — curated history

## Develop & release

`helm lint` + `helm template` on a placeholder-substituted copy (see Install above);
PRs touching `apis/**`/`configmaps/**` are statically validated by
[pr-ci.yaml](.github/workflows/pr-ci.yaml). Release runbook: [docs/release.md](docs/release.md).
