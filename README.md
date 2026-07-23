# krateo-oas

OpenAPI → Kubernetes-kind GitOps publish path for the Krateo **API Builder** (KOG).

The portal's API Builder (Autopilot rail) authors a `RestDefinition` (ogen.krateo.io) from a
pasted OpenAPI document and opens a **pull request** against this repo — one PR per API kind, on
a `builder/<kind>` branch — instead of writing the CR straight to the cluster. Each PR carries:

```
apis/<kind>/restdefinition.yaml      # the RestDefinition CR (kind, group, verbs → the generated K8s kind)
apis/<kind>/openapi.yaml             # (paste-case) the source OpenAPI document, held verbatim
```

## The flow

1. **Author** — API Builder proposes the RestDefinition; the human reviews the full YAML at a
   blast-radius gate in the rail.
2. **PR** — a `builder/<kind>` branch + PR lands here (github.krateo.io provider CRs).
3. **Review & merge** — a human merges. Nothing reaches the cluster before this gate.
4. **Reconcile** *(the GitOps apply — see "Status" below)* — the merged `apis/<kind>/*` is applied
   to the cluster; the oasgen/KOG controller generates the CRD + controller and the new kind
   (`Refund · billing.acme.io`, etc.) appears Ready in the registry.

## Status

Steps 1–3 are live (portal ≥ 1.5.72 / frontend ≥ 1.3.77). **Step 4 (merge → live kind) is not yet
wired**: it needs a GitOps apply mechanism (CI that `kubectl apply`s the merged `apis/**` manifests,
or a Flux/Argo/rest-dynamic-controller watch on this repo). Until that exists, a merged PR does not
auto-materialise the kind — apply the RestDefinition manually, or add the apply step. See the portal
memory note `ux-round2-punchlist` for the decision context.
