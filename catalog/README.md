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
| Merchandising project | `merchandising` | project merchandising and its components (ad, recommendation; shoppingassistant when it lands) |
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

## Before the portal will show any of this

These files need to be ingested. The showroom's catalog is populated from the OpenChoreo API by
a catalog backend module, so a static location has to be registered with the portal before
`catalog/` is read at all. That is portal configuration and sits installer-side, not here.

**Unverified:** whether the generated entities for projects, components and resources can carry
an owner that points at these groups. The epic asks for ownership down to resource level, and
the mapping above records the intent, but whether the catalog module propagates an owner onto
the entities it generates was not confirmed. Check before promising the ownership arrows appear
in the portal.
