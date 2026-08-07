---
type: ExampleIndex
title: krateo-oas — examples
description: Index of the runnable examples under examples/ — currently one complete API-Builder contribution you can validate and render locally.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [examples]
timestamp: 2026-08-07T00:00:00Z
---

# Examples

- [echo-api](../examples/echo-api/README.md) — a complete, valid API-Builder
  contribution (`Echo` · `demo.krateo.io`: RestDefinition + OAS ConfigMap). No cluster
  needed: copy the pair into `apis/`/`configmaps/`, then lint + render the chart
  locally to see exactly what CDC would apply; propose it for real by PR.

The live seed API in the repo itself — [`apis/refund`](../apis/refund/restdefinition.yaml)
+ [`configmaps/refund-oas.yaml`](../configmaps/refund-oas.yaml) (`Refund` ·
`billing.acme.io`) — doubles as a merged, end-to-end reference.
