# krateo-oas

OpenAPI → Kubernetes-kind GitOps publish path for the Krateo **API Builder** (KOG).

This repo is **also a Helm chart**: its root is a singleton "API registry" chart that the Krateo
composition-dynamic-controller (CDC) renders. Merged API-Builder artifacts under `apis/**` and
`configmaps/**` become **live cluster resources** — a merged PR is a live kind. The human PR merge
is the **hard gate**: nothing reaches the cluster before it.

```
apis/<kind>/restdefinition.yaml      # the RestDefinition CR (kind, group, verbs → the generated K8s kind)
configmaps/<kind>-oas.yaml           # a ConfigMap manifest (name: <kind>-oas) holding the OpenAPI doc,
                                     # referenced by the RestDefinition's oasPath configmap:// URL
Chart.yaml, values.yaml, values.schema.json, templates/registry.yaml   # the registry chart
```

`templates/registry.yaml` globs `configmaps/*-oas.yaml` + `apis/**/restdefinition.yaml` and emits
each verbatim, so CDC **owns, drift-heals, and prunes** them natively — no bespoke controller, no
in-cluster poll loop.

## The flow

1. **Author** — the API Builder (Autopilot rail) proposes the `RestDefinition` + OAS `ConfigMap`;
   the human reviews the full YAML at a blast-radius gate in the rail.
2. **PR** — a `builder/<kind>` branch + PR lands here, authored via the github.krateo.io provider CRs
   (`RepoContent`/`PullRequest`). `pr-ci.yaml` statically validates each proposed RestDefinition.
3. **Review & merge** — a human merges. **This is the hard gate** — nothing reaches the cluster before it.
4. **Release** — on merge to `main`, `release-oci.yaml` packages + pushes the chart to
   `oci://ghcr.io/krateo-platformops/charts/oas:<tag>`, and the `krateo-oas` installer-component pin
   is bumped to `<tag>` (a git commit — the durable component-pins path; no `kubectl apply`).
5. **Materialise** — CDC pulls the new pinned chart version and renders it → the `RestDefinition`
   + OAS `ConfigMap` go live → oasgen-provider generates the CRD + controller and the new kind
   (`Refund · billing.acme.io`, etc.) appears Ready. A merged deletion PR → next version → CDC prunes.

## Why a chart, not a poller

CDC renders a Composition from a **pinned chart version** and can't chase a git branch, so the
merge→live trigger is the release step (step 4), not an in-cluster watch. That keeps the whole loop
inside native CDC semantics — apply, self-heal, and prune are the controller's job — and adds no
runtime component. The only new automation is standard release plumbing (tag + publish + pin bump),
and the human merge stays the authoritative gate.
