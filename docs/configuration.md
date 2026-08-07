---
type: Configuration
title: krateo-oas — configuration
description: The values surface of the singleton registry chart — deliberately empty; the chart's real content is the merged apis/** and configmaps/** files, not values.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [configuration, values]
timestamp: 2026-08-07T00:00:00Z
---

# Configuration

krateo-oas is a **singleton registry chart with no per-instance configuration** — one
honest fact, grounded in [`values.yaml`](../values.yaml) and
[`values.schema.json`](../values.schema.json):

| Value | Type | Default | Effect |
|---|---|---|---|
| `global` | object | `{}` | Helm globals passthrough (injected when used as a subchart). It exists **only** so CDC generates a non-empty (but trivial) CRD from `values.schema.json`. No template reads it. |

`values.schema.json` sets `additionalProperties: false`, so any other value is rejected.
There are no environment variables, no image, and no tunables: `appVersion` tracks the
chart version because the chart ships no application.

The chart's real "configuration" is its **content** — the merged
`apis/<kind>/restdefinition.yaml` and `configmaps/<kind>-oas.yaml` files that
[`templates/registry.yaml`](../templates/registry.yaml) renders verbatim. Changing what
the chart deploys means merging a PR, not setting values: see [usage.md](./usage.md)
and the file contract in [api.md](./api.md).
