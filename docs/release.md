---
type: Runbook
title: krateo-oas — release
description: How a release actually ships — plain-semver git tag runs the canonical release-oci workflow, which packages the registry chart and pushes it to GHCR; the consuming pin is then bumped by git commit.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [release, oci, ghcr]
timestamp: 2026-08-07T00:00:00Z
---

# Release

A release is what turns merged content into something CDC can materialise — for this
repo, tagging **is** the deploy trigger (step 4 of the flow in
[overview.md](./overview.md)).

## Mechanics (grounded in `release-oci.yaml`)

- **Trigger**: a plain-semver git tag `X.Y.Z` — **no `v` prefix** (the tag filter is
  `[0-9]+.[0-9]+.[0-9]+`). `workflow_dispatch` with an explicit `chart_version` also
  works.
- **Workflow**: [`.github/workflows/release-oci.yaml`](../.github/workflows/release-oci.yaml),
  the byte-identical canonical org workflow. It discovers the single first-class chart
  (the repo root), substitutes the `CHART_VERSION` placeholder in
  [`Chart.yaml`](../Chart.yaml) with the tag (`appVersion` uses the same placeholder —
  there is no application image, so app version tracks chart version), packages, and
  pushes to:

  ```
  oci://ghcr.io/krateo-platformops/charts/krateo-oas:<tag>
  ```

- **Packaged content**: [`.helmignore`](../.helmignore) excludes repo scaffolding but
  deliberately **keeps `apis/` and `configmaps/`** — the template reads them via
  `.Files.Glob`, so they must ship inside the chart.
- **Pin bump**: publishing alone changes nothing in-cluster. The consuming pin — the
  deployment-side installer `components` append or the standalone
  `CompositionDefinition` `spec.chart.version` (see [usage.md](./usage.md)) — must be
  bumped to `<tag>`. This is a git commit (the durable pin path), not a
  `kubectl apply` against live composition internals. It is not automated by this
  repo's workflows.

## Verify after tagging

```sh
helm pull oci://ghcr.io/krateo-platformops/charts/krateo-oas --version <tag>
tar tzf krateo-oas-<tag>.tgz | grep -E 'apis/|configmaps/'
```

## Version history reality

The only existing tag, `0.1.0` (2026-07-23), **predates the org migration**: at that
tag the workflow pushed to the legacy pre-migration user registry, so
`ghcr.io/krateo-platformops/charts/krateo-oas` has **no published version yet**
(verified 2026-08-07: `helm pull --version 0.1.0` → not found). The next tag will be
the first publish to the org registry.
