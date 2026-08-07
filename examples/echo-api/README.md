---
type: Example
title: krateo-oas — echo-api example
description: A complete, valid API-Builder contribution (Echo · demo.krateo.io) — copy the pair into apis/ and configmaps/, then lint and render the registry chart locally to see exactly what CDC would apply.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [example, restdefinition]
timestamp: 2026-08-07T00:00:00Z
---

# echo-api example

A ready-made contribution pair satisfying the [file contract](../../docs/api.md):

- [`restdefinition.yaml`](./restdefinition.yaml) — `Echo` in group `demo.krateo.io`
  (create/get/delete verbs, identifier `id`), destined for
  `apis/echo/restdefinition.yaml`.
- [`echo-oas.yaml`](./echo-oas.yaml) — the `echo-oas` ConfigMap holding the OpenAPI
  document under `openapi.yaml`, destined for `configmaps/echo-oas.yaml`.

**Preconditions**: `git`, `helm`. No cluster — this validates and renders locally;
going live is always a PR merge + release (the hard gate), never a direct apply.

From this directory, stage the pair in a scratch copy of the repo, substitute the
`CHART_VERSION` placeholder (not valid semver, so `helm` rejects the raw checkout),
then lint + render:

```sh
rm -rf /tmp/krateo-oas-echo
cp -R ../.. /tmp/krateo-oas-echo
mkdir -p /tmp/krateo-oas-echo/apis/echo
cp restdefinition.yaml /tmp/krateo-oas-echo/apis/echo/restdefinition.yaml
cp echo-oas.yaml /tmp/krateo-oas-echo/configmaps/echo-oas.yaml
sed -i '' 's/CHART_VERSION/0.0.0-local/g' /tmp/krateo-oas-echo/Chart.yaml   # Linux: sed -i
helm lint /tmp/krateo-oas-echo
helm template krateo-oas /tmp/krateo-oas-echo
```

The render lists the `echo-oas` ConfigMap and the `echo` RestDefinition alongside the
merged `refund` seed pair (ConfigMaps first, then RestDefinitions, alphabetical within
each glob) — verbatim, each annotated with its source path. To propose
it for real, commit the two files at their destinations on a `builder/echo` branch and
open a PR; [`pr-ci.yaml`](../../.github/workflows/pr-ci.yaml) runs the same contract
checks in CI.
