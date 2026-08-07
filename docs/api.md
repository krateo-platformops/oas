---
type: API
title: krateo-oas — api
description: The contract this repo exposes — the resources the chart emits (RestDefinitions + OAS ConfigMaps) and the file contract every contribution must satisfy, as enforced by pr-ci.yaml.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [api, restdefinition, configmap, contract]
timestamp: 2026-08-07T00:00:00Z
---

# API

This repo exposes no HTTP endpoints and owns no CRDs of its own. Its contract is
twofold: **what the chart emits** into the cluster, and **the file contract** a
contribution PR must satisfy.

## What the chart emits

[`templates/registry.yaml`](../templates/registry.yaml) renders, verbatim and in this
order:

1. every `configmaps/*-oas.yaml` — a `v1/ConfigMap` named `<kind>-oas` whose
   `data."openapi.yaml"` is the OpenAPI document;
2. every `apis/**/restdefinition.yaml` — an `ogen.krateo.io/v1alpha1 RestDefinition`.

Downstream, [oasgen-provider](https://github.com/krateo-platformops/oasgen-provider)
turns each RestDefinition into a generated CRD (`<resource.kind>` in group
`<resourceGroup>`, e.g. `Refund` · `billing.acme.io`) plus its controller
(rest-dynamic-controller). Those generated kinds are the platform-visible API surface
this registry produces — but they are owned and documented by oasgen-provider.

## The file contract for contributions

One directory per kind, two files, no namespace in either manifest (CDC applies both
into the composition's namespace, `krateo-system`):

```
apis/<kind>/restdefinition.yaml
configmaps/<kind>-oas.yaml
```

[`pr-ci.yaml`](../.github/workflows/pr-ci.yaml) statically enforces, per
`apis/<kind>/restdefinition.yaml`:

| Check | Requirement |
|---|---|
| YAML | a single valid YAML document |
| `kind` | `RestDefinition` |
| `apiVersion` | `ogen.krateo.io/*` |
| `spec.oasPath` | required |
| `spec.resourceGroup` or `spec.resource` | at least one (the generated kind) |
| paired ConfigMap | if `spec.oasPath` is a `configmap://` URL, `configmaps/<kind>-oas.yaml` must exist in the PR |

The `oasPath` convention is `configmap://krateo-system/<kind>-oas/openapi.yaml`, and
the ConfigMap must be named `<kind>-oas` with the document under the `openapi.yaml`
key. `spec.resource` declares the generated kind, its identifiers, and its
`verbsDescription` (action → HTTP method + path) — see the seed
[`apis/refund/restdefinition.yaml`](../apis/refund/restdefinition.yaml) for the full
shape, and the RestDefinition CRD reference in oasgen-provider for every field.
