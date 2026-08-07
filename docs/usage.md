---
type: Usage
title: krateo-oas — usage
description: How the registry chart is consumed (CompositionDefinition / installer component append), how to add or remove an API by PR, and the local placeholder-substituted render recipe.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [usage, cdc, installer, helm]
timestamp: 2026-08-07T00:00:00Z
---

# Usage

## How it reaches a cluster

krateo-oas is not installed by hand: CDC renders it from a **pinned chart version**.
Two equivalent wirings:

1. **Installer component append** (the durable pin path): add a `krateo-oas` entry to
   the installer's additive `components` value; the installer registers the
   CompositionDefinition and manages the Composition CR lifecycle with the rest of the
   platform. As of installer main, `krateo-oas` is **not** in the stock
   `files/component-pins.yaml` — the append is deployment-side.
2. **Standalone CompositionDefinition** on any cluster running core-provider:

```yaml
apiVersion: core.krateo.io/v1alpha1
kind: CompositionDefinition
metadata:
  name: krateo-oas
  namespace: krateo-system
spec:
  chart:
    url: oci://ghcr.io/krateo-platformops/charts/krateo-oas
    version: "<tag>"
```

core-provider generates a `KrateoOas` composition kind (crdgen PascalCase of the chart
name, API version derived from the chart version, e.g. `composition.krateo.io/v0-1-0`);
creating one CR of it in `krateo-system` makes CDC install the chart there — which is
exactly what the `configmap://krateo-system/...` `oasPath` URLs expect. Registry
publish status: see [release.md](./release.md).

## Adding an API (the normal path)

1. Add `apis/<kind>/restdefinition.yaml` + `configmaps/<kind>-oas.yaml` satisfying the
   contract in [api.md](./api.md) (the API Builder rail authors these for you on a
   `builder/<kind>` branch).
2. Open a PR; `pr-ci.yaml` validates it statically.
3. A human reviews and merges — the hard gate.
4. Tag a release and bump the consuming pin ([release.md](./release.md)); CDC
   materialises the new kind.

Removing an API is the same loop with a deletion PR: on the next pinned version CDC
**prunes** the RestDefinition + ConfigMap, and oasgen-provider tears down the kind.

## Local render (no cluster)

`Chart.yaml` carries the `CHART_VERSION` placeholder (substituted at tag time), which
is not valid semver, so `helm template` needs a substituted copy:

```sh
git clone https://github.com/krateo-platformops/oas.git
cp -R oas /tmp/krateo-oas-render
sed -i '' 's/CHART_VERSION/0.0.0-local/g' /tmp/krateo-oas-render/Chart.yaml   # Linux: sed -i
helm lint /tmp/krateo-oas-render
helm template krateo-oas /tmp/krateo-oas-render
```

The output is every merged OAS ConfigMap followed by every RestDefinition, each
annotated with its source path — exactly what CDC will apply. To pull a released chart
instead: `helm pull oci://ghcr.io/krateo-platformops/charts/krateo-oas --version <tag>`.

## Trying an end-to-end contribution

[examples/echo-api](../examples/echo-api/README.md) ships a complete valid pair
(RestDefinition + OAS ConfigMap) with the copy → validate → render commands.
