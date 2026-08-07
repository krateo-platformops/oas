---
type: Log
title: krateo-oas — log
description: Curated chronological history of the API Builder GitOps publish target — notable changes and decisions, newest first.
resource: oci://ghcr.io/krateo-platformops/charts/krateo-oas
tags: [log, history]
timestamp: 2026-08-07T00:00:00Z
---

# Log

## 2026-08-07
- Adopted the Krateo Documentation Standard (this bundle) and wired the shared
  `lint-docs` conformance job into PR CI.

## 2026-08-03
- Org migration: CI caller, OCI registry and links re-pointed from the legacy user
  account to `krateo-platformops` (`7d1999b`). Charts now publish to
  `oci://ghcr.io/krateo-platformops/charts`; tag `0.1.0` predates this, so the org
  registry carries no published version until the next tag (see
  [release.md](./release.md)).

## 2026-07-23
- **Decision: registry chart, not a poller** — the repo root became a CDC-rendered
  singleton chart (`f906b1b`, tagged `0.1.0`): `templates/registry.yaml` globs merged
  `apis/**` + `configmaps/**` and emits each verbatim, so CDC owns/heals/prunes
  natively and the merge→live trigger is the release step. Rationale in
  [overview.md](./overview.md).
- Seeded the `Refund` (`billing.acme.io`) RestDefinition + OAS ConfigMap to exercise
  the oas-gitops loop end-to-end (`46c9748`).
- Added `pr-ci.yaml`: static validation of proposed RestDefinitions, parity with the
  krateo-blueprints PR gate (`8382aec`).
- Repo seeded as the API Builder GitOps publish target (`447ee71`).
