---
title: Internal Developer Platforms - Why We Chose CNCF Meshery
publishDate: 06-27-2025
author: Hamza Mohd
description: |
    Notes on building an internal developer platform with CNCF Meshery as the operational core.
keywords: "internal developer platform, idp, platform engineering, meshery, cncf, kubernetes, devops"
layout: "../../../layouts/BaseLayout.astro"
---

For the past year, a small group of us has been quietly building an internal developer platform (IDP) for engineering teams to ship services without having to learn every corner of Kubernetes. The pitch is familiar: golden paths, paved roads, self-service. The reality is harder. An IDP that nobody uses is just another internal tool with a wiki page that goes stale.

After trying a few combinations, the operational core we kept coming back to is [CNCF Meshery](https://meshery.io). This post is a short, opinionated note on why.

### What we wanted from the platform

We did not set out to build "Heroku, but internal." We wanted something narrower and more honest:

- A single place where a service owner can see what is deployed, where, and at which version.
- A way to compose infrastructure (a service, its config, its policies, its add-ons) as one unit rather than a pile of disjoint YAML.
- A way to make changes with a meaningful preview, not just `kubectl apply` and hope.
- A way to bring in service mesh, observability, and policy without each becoming its own snowflake project.

Most platforms answer one or two of these. The gap is usually the second one: the unit of work. Teams end up reasoning in raw Kubernetes objects, which is the wrong altitude for an IDP.

### Why Meshery fit

Meshery does a few things that lined up well with how we wanted teams to work.

**Designs as the unit of deployment.** A Meshery Design is a composable, declarative artifact - a service plus its dependencies plus its operational concerns. It maps cleanly to how product teams already think about "the thing I own." We could finally stop saying "your service is these seven manifests across three repos."

**A real catalog, not a list of links.** The catalog gives platform teams a place to publish vetted patterns. When a new team starts, they pick a design, fork it, and they are off. The platform shows up as something usable, not just a Confluence page.

**Operator-grade lifecycle management.** Meshery runs as an operator inside our clusters and tracks the state of what is deployed against what is defined. Drift, partial rollouts, and unmanaged resources become visible instead of folklore.

**Service mesh and add-ons without lock-in.** We are a mixed environment (Istio in one region, Linkerd in another, plus a long tail of operators). Meshery treats meshes and add-ons as first-class but pluggable. We did not have to standardize the mesh before standardizing the platform, which would have been a multi-quarter blocker.

**CNCF governance.** This matters more than it sounds. Adopting a project under the CNCF means a public roadmap, a real community, and a vendor-neutral home. For an internal platform that we want to outlive any single decision, that lineage was a non-trivial factor.

### What changed for teams

The shift was not dramatic on day one. It rarely is. What we noticed over a few months:

- New services going from "ticket filed" to "running in staging" dropped from days to hours.
- Onboarding a new engineer to a service became "open the design" instead of "let me walk you through the repos."
- Platform changes (a new default policy, a new sidecar version) went out as design updates rather than a fleet-wide migration script.
- Incident reviews started referencing the design history. That alone was worth the migration.

### What we are still figuring out

Meshery is not a silver bullet, and I want to be honest about the rough edges.

- The mental model takes a session or two to click for engineers used to raw manifests. We wrote a short internal guide and ran a couple of brown bags. After that, it stuck.
- Designing for multi-tenancy across a large org needs deliberate thought. We sketched our boundaries (orgs, environments, ownership) before we onboarded the second team, and I would do that again.
- Policy and governance are evolving quickly upstream. We track releases and budget time to keep up. Worth it, but real work.

### Why not just X?

We looked at the usual alternatives. Some are good products. Most of them either pulled us toward a specific opinionated stack we did not want, or they solved the developer-facing surface beautifully but left the operational core thin. The combination of "composable unit of work" plus "operator that actually reconciles" plus "neutral, community-governed" was the thing we could not replicate.

### Takeaway

If you are building an IDP and you have not looked at Meshery in a while, look again. The simplest version of our story is this: we stopped trying to glue platform pieces together and started shipping designs. The platform got smaller and more useful at the same time, which is usually the sign that you picked the right primitive.
