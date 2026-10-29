# `pending/` — scaffolded, deliberately not applied

No Flux Kustomization points at this directory, and `kubectl apply -R -f namespaces/` does not
reach it. Everything here is structurally complete and blocked on something that is **not
configuration**: either an image that does not exist yet, or a ClusterTrait that is not installed
on any cluster.

They live here rather than in `namespaces/` because the alternative is worse. A component with a
`REPLACE-ME` image still lands in its project's release and crash-loops in every bound
environment — `spec.autoDeploy` does not gate that; it only controls automatic deployment on
creation. Keeping them out of the applied set means the repo describes the whole scenario while
the cluster only runs what can actually run.

## What is here, and what unblocks it

| Scaffold | Blocked on |
|---|---|
| `merchandising/components/shoppingassistant/` | (a) a forked image — upstream v0.10.5 publishes **no** `shoppingassistantservice` image, verified against `release/kubernetes-manifests.yaml` at that tag; (b) the `ai-llm-routing` ClusterTrait, which ships with the AI gateway module, not OpenChoreo 1.3.1; (c) the open fork-vs-simpler-NL-search decision (`GCP-DEMO-IMPLEMENTATION.md` 0.2) |

Neither AI trait exists in `openchoreo/openchoreo@v1.3.1` (`install/`, `samples/` — checked).
`AI-GATEWAY-INSTALL-PLAN.md` in the task directory is still *"Nothing applied. Awaiting
approval."*, and its step 4 must also patch the `service` ClusterComponentType's `allowedTraits`
— otherwise the traits exist but no component is permitted to carry them.

## Activating

Paths mirror their destinations exactly, so activation is a move and nothing else.

**Insights project — SHIPPED 2026-10-29.** Both images were built
(`v1-3676cf0d`) and the project now lives at
`namespaces/boutique/projects/insights/`. It is bound to `development` only;
the four other binding files travelled with it, gated and inert, and carry
their own reasoning. Nothing in this section applies to it any more.

**Shopping assistant** — once the fork is built and the gateway module is installed:

```sh
git mv pending/namespaces/boutique/projects/merchandising/components/shoppingassistant \
       namespaces/boutique/projects/merchandising/components/shoppingassistant
```

Then wire the front end, which is the only edit to an already-applied file. `frontend`'s workload
currently carries the two upstream opt-out values:

```yaml
    - key: SHOPPING_ASSISTANT_SERVICE_ADDR
      value: '-'
    - key: ENABLE_ASSISTANT
      value: 'false'
```

Replace the first with a real dependency entry and flip the second:

```yaml
  dependencies:
    endpoints:
    - project: merchandising
      component: shoppingassistant
      name: http
      visibility: organization
      envBindings:
        address: SHOPPING_ASSISTANT_SERVICE_ADDR
  container:
    env:
    - key: ENABLE_ASSISTANT
      value: 'true'
```

That makes a **fifth** cross-project edge — `frontend` (storefront) → `shoppingassistant`
(merchandising) — so the "four cross-project edges" section of the top-level `README.md` needs
updating at the same time.

In both cases remove the `NOT APPLIED` header comment from each file as you move it.

## What will never live here

The AI gateway itself. It is platform infrastructure in `openchoreo-data-plane`, outside all five
projects, installed by Helm and owned by no project. The `ai-llm-routing` / `ai-mcp-federation`
traits attach to the **consuming** component, which is why they are declared on
`shoppingassistant` and `insights-api` above and not on anything resembling a gateway component.
