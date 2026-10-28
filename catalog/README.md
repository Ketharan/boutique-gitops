# Catalog: teams and people

Five teams, three people each, fifteen in all. They exist so the portal can show **who owns
what**. Nothing here is an identity.

## These are not Kubernetes objects

No Kustomization points at `catalog/`, and `kubectl apply` would reject these files: they are
Backstage catalog entities, not cluster resources. They are kept out of every apply path on
purpose, the same way `pending/` is.

## They are display metadata, not access control

The showroom is read-only and its visitors are anonymous. There are no Thunder accounts behind
these people, no per-user authorization bindings, and no login. What a visitor may see is
governed entirely by the `kiosk-viewer` role, which is unchanged by anything in this directory.
If ownership should ever drive authorization, that is a separate and much larger decision.

## Nothing here is real

Every name is invented and role-shaped rather than person-shaped, and every address is on
`boutique.example.com`. `example.com` is reserved by RFC 2606 for documentation and examples, so it cannot
collide with a real domain. No real person, company or employer appears anywhere in this
directory, which matters because this repository is public and meant to be cloned.

## Who owns what

| Team | Group | Owns |
|---|---|---|
| Storefront | `storefront` | project storefront, its 7 components, and the cart-cache Resource |
| Merchandising Team | `merchandising` | project merchandising and its components (ad, recommendation; shoppingassistant when it lands) |
| Notifications | `notifications` | project notifications and its email component |
| Insights | `insights` | project insights and its 2 components (not yet applied: pending/) |
| Traffic Simulator | `traffic-sim` | project traffic-sim and its loadgenerator component, in the quality-engineering org unit |

## Two things that look like faults and are not

**Two of the five teams have nothing to own yet.** `Insights` is still scaffolded in `pending/`
and `Traffic Simulator` lives in the `quality-engineering` org unit, which is complete in this
repository but not yet applied to the cluster. Their people can be authored now; they will
simply have nothing attached until those land.

**Entities do not appear immediately.** The portal's catalog converges on a five-minute poll;
event-driven ingestion is deliberately disabled for the public kiosk. An entity missing a few
seconds after a change is the poll interval, not a failure.

## Ownership propagates. These files are the wrong vehicle for the people.

Both halves were settled on 2026-10-27 by reading the portal's catalog modules
(`openchoreo/backstage-plugins`), and the answers differ.

**Ownership: yes, and it is already wired.** `resolveComponentOwner`
(`catalog-backend-module-openchoreo/src/utils/helpers.ts:39-57`) resolves an entity's owner as
*component annotation > project annotation > the portal's configured default*, reading
`backstage.io/owner` off the OpenChoreo CR. So each `project.yaml` in this repository now carries
`backstage.io/owner: group:default/<team>`, and every component in that project inherits it
unless a component overrides it. `default` there is the **Backstage** namespace, not the
Kubernetes one.

**The people: not from these files.** Users and Groups in this portal come from **Thunder IdP**,
synced by a separate module — `catalog-backend-module-openchoreo-users`, described by its own
README as a "Thunder User & Group Entity Provider" performing scheduled full syncs. It is not a
static-location reader. So registering `catalog/` as a catalog location is not how this portal
gets its teams, and these two files describe the right org chart in a shape this portal does not
consume.

That leaves a decision rather than a task: create the five groups and fifteen people in Thunder
so the provider syncs them, or keep these files as the written-down answer to "who owns what"
and accept that the portal's Teams page is populated by a different mechanism. The first means
creating fifteen IdP identities for fictional people on a showroom whose visitors are anonymous,
which is a bigger step than catalog metadata and worth a deliberate decision.

Until groups exist by either route, the `backstage.io/owner` annotations resolve to group
references that do not yet exist. That is harmless — it is the same state as an unannotated
entity falling back to the default owner — and it means the annotations can land now and start
working the moment the groups do.
