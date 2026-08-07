---
type: Architecture
title: krateo-oas — overview
description: How the API Builder GitOps loop works — a merged RestDefinition PR becomes a live Kubernetes kind through a released registry chart that CDC renders, with no bespoke controller.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [api-builder, gitops, cdc, oasgen-provider]
timestamp: 2026-08-07T00:00:00Z
---

# Overview

krateo-oas closes the loop **OpenAPI document → reviewed PR → live Kubernetes kind**.
It is simultaneously a git publish target (the API Builder opens PRs against it) and a
Helm chart (CDC renders whatever is merged). There is no application image and no
runtime component of its own.

## Repo layout

```
apis/<kind>/restdefinition.yaml      # the RestDefinition CR (group, kind, verbs → the generated K8s kind)
configmaps/<kind>-oas.yaml           # a ConfigMap manifest (name: <kind>-oas) holding the OpenAPI doc,
                                     # referenced by the RestDefinition's oasPath configmap:// URL
Chart.yaml, values.yaml,
values.schema.json,
templates/registry.yaml              # the singleton registry chart
```

## The flow

1. **Author** — the API Builder (Autopilot rail) proposes the `RestDefinition` + OAS
   `ConfigMap`; the human reviews the full YAML at a blast-radius gate in the rail.
2. **PR** — a `builder/<kind>` branch + PR lands here, authored via the
   github.krateo.io provider CRs (`RepoContent`/`PullRequest`).
   [`pr-ci.yaml`](../.github/workflows/pr-ci.yaml) statically validates each proposed
   RestDefinition (see [api.md](./api.md) for the exact checks). CI has **no cluster
   credentials** — applying to the cluster is never a push from CI.
3. **Review & merge** — a human merges. **This is the hard gate**: nothing reaches the
   cluster before it.
4. **Release** — a plain-semver tag runs
   [`release-oci.yaml`](../.github/workflows/release-oci.yaml), which packages the chart
   and pushes it to `oci://ghcr.io/krateo-platformops/charts/krateo-oas:<tag>`; the
   consuming pin (installer component append or standalone `CompositionDefinition`) is
   then bumped to `<tag>` — a git commit, the durable pin path, not a `kubectl apply`.
   See [release.md](./release.md).
5. **Materialise** — CDC pulls the newly pinned chart version and renders it: the
   `RestDefinition` + OAS `ConfigMap` go live, oasgen-provider generates the CRD +
   controller, and the new kind (`Refund` · `billing.acme.io`, etc.) appears Ready.
   A merged **deletion** PR → next version → CDC **prunes** the resources.

## The registry-chart mechanics

[`templates/registry.yaml`](../templates/registry.yaml) is the entire template surface:
two `.Files.Glob` loops that emit each merged file **verbatim** —
`configmaps/*-oas.yaml` first, then `apis/**/restdefinition.yaml`. Because the artifacts
are chart resources, CDC **owns, drift-heals and prunes** them natively; no bespoke
controller, no in-cluster poll loop. Notes traced to the template:

- **Ordering is a courtesy, not a requirement** — oasgen-provider reconciles a
  RestDefinition asynchronously and retries until its ConfigMap exists.
- **No namespaces in the manifests** — CDC applies both into the composition's
  namespace (`krateo-system`), which is exactly what the
  `configmap://krateo-system/<kind>-oas/openapi.yaml` `oasPath` references.
- **`.helmignore` must not ignore `apis/` or `configmaps/`** — the template reads them
  via `.Files.Glob`, so they are packaged into the chart (only repo scaffolding —
  `.git/`, `.github/`, `*.md`, `dist/` — is excluded).

## Why a chart, not a poller

CDC renders a Composition from a **pinned chart version** and can't chase a git branch,
so the merge→live trigger is the release step (step 4), not an in-cluster watch. That
keeps the whole loop inside native CDC semantics — apply, self-heal and prune are the
controller's job — and adds no runtime component. The only automation is standard
release plumbing (tag + publish + pin bump), and the human merge stays the
authoritative gate.

## Boundaries

- **Upstream**: the API Builder rail (Autopilot) authors the PRs; humans merge them.
- **Downstream**: CDC (chart render/heal/prune) and
  [oasgen-provider](https://github.com/krateo-platformops/oasgen-provider) (turns each
  RestDefinition into a CRD + controller). The OpenAPI documents in this repo are inert
  data here; the OpenAPI parsing machinery — including the Krateo-maintained fork of the
  upstream `pb33f` libopenapi Go library that oasgen-provider pins via a `go.mod`
  `replace` — lives entirely in that repo, not this one.
