---
type: ChartRepo
title: krateo-oas — index
description: The map of the krateo-oas doc bundle — the API Builder GitOps publish target, a singleton registry chart CDC renders so a merged RestDefinition PR becomes a live Kubernetes kind.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [api-builder, kog, gitops, restdefinition, oasgen-provider]
timestamp: 2026-08-07T00:00:00Z
---

# krateo-oas

krateo-oas is the **GitOps publish target of the Krateo API Builder** (KOG). The repo
root is a singleton "API registry" Helm chart: `templates/registry.yaml` globs the merged
`apis/**/restdefinition.yaml` and `configmaps/*-oas.yaml` files and emits each verbatim,
so the composition-dynamic-controller (CDC) applies, drift-heals and prunes them natively.
A merged PR → a chart release → CDC renders the new pin → oasgen-provider generates the
CRD + controller → the new kind is live. The human PR merge is the hard gate.

Despite the name, this repo carries **no Go code** and no OpenAPI tooling: the OpenAPI
documents stored here are inert data. Parsing them happens downstream in
[oasgen-provider](https://github.com/krateo-platformops/oasgen-provider).

## The bundle (start here)

- [overview](./overview.md) — the author → PR → merge → release → materialise flow, the
  registry-chart mechanics, and why it's a chart, not a poller.
- [usage](./usage.md) — how CDC and the installer consume the chart; how to add or
  remove an API; local render recipe.
- [configuration](./configuration.md) — the values surface (deliberately empty: a
  singleton with no per-instance configuration).
- [api](./api.md) — the file contract a contribution must satisfy (enforced by
  `pr-ci.yaml`) and the resources the chart emits.
- [examples](./examples.md) — the runnable example under `examples/`.
- [release](./release.md) — how a release ships: plain-semver tag → OCI chart on GHCR →
  installer-side pin bump.
- [log](./log.md) — curated history.
- [llms.txt](./llms.txt) — the version-pinned agent index of this bundle.

## The moving parts (source of truth)

- [`templates/registry.yaml`](../templates/registry.yaml) — the whole template surface
  (two `.Files.Glob` loops).
- [`apis/refund/restdefinition.yaml`](../apis/refund/restdefinition.yaml) +
  [`configmaps/refund-oas.yaml`](../configmaps/refund-oas.yaml) — the seed API
  (`Refund` · `billing.acme.io`) that exercises the loop end-to-end.
- [`.github/workflows/pr-ci.yaml`](../.github/workflows/pr-ci.yaml) — the static PR gate.
- [`.github/workflows/release-oci.yaml`](../.github/workflows/release-oci.yaml) — the
  canonical org release workflow.
