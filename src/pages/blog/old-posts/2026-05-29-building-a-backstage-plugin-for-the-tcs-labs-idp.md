---
title: Building a Backstage Plugin for the TCS Labs Internal Developer Platform
publishDate: 05-29-2026
author: Hamza Mohd
description: |
    What it took to build a Backstage plugin that made the TCS Labs internal developer platform easier to discover, navigate, and operate.
keywords: "backstage, backstage plugin, internal developer platform, idp, platform engineering, developer portal, tcs labs"
layout: "../../../layouts/BaseLayout.astro"
---

The hardest part of an internal developer platform is usually not the platform logic. It is the gap between what the platform can do and what engineers can actually find, trust, and use without opening three tabs and asking someone on Slack.

That was the reason we started building a Backstage plugin for the TCS Labs internal developer platform. We already had working platform primitives: service templates, deployment workflows, ownership metadata, environment-specific controls, and a growing set of paved-road conventions. What we did not have was a surface that made those capabilities feel coherent to the engineers consuming them.

Backstage was the obvious place to meet them. It was already where service catalogs, documentation, and team context lived. The missing piece was a plugin that treated the internal platform as a first-class product instead of a loose collection of links.

### What the plugin needed to do

We kept the initial brief narrow.

- Show the operational context of a service without forcing teams to jump across tools.
- Expose golden-path actions in the same place engineers already look up service metadata.
- Make platform ownership visible: who owns the service, which environment it is deployed to, and which workflows are expected.
- Stay opinionated enough to be useful without turning Backstage into a second control plane.

That last constraint mattered. A portal that tries to own every write path usually becomes slow, fragile, and politically difficult to evolve. We wanted a plugin that provided context, entry points, and a few well-chosen workflows - not a giant abstraction layer over the entire platform.

### The data model came first

The first real decision was not UI. It was data shape.

Most Backstage plugins get awkward when they start by rendering whatever fields happen to exist today. We tried to avoid that by defining the service-level questions we wanted the page to answer:

- What is this service?
- Who owns it?
- Where is it running?
- Which platform capabilities are enabled for it?
- What are the safe next actions?

Once those questions were clear, the plugin API became easier to design. We normalized the inputs from our catalog metadata, internal platform services, and deployment records into one service view model. That let the UI stay boring in the good way: predictable cards, stable sections, and fewer "if this field exists" branches.

### The best UI work was removing choices

One of the recurring mistakes in internal portals is making every team learn the platform's internal taxonomy. Engineers do not want to think in terms of "provisioning subsystem," "deployment orchestrator," or "policy attachment workflow." They want to know whether their service is healthy, what is configured, and what they can do next.

So the plugin UI centered on a few things:

- A service summary that established ownership and runtime context quickly.
- A capabilities section that showed which platform features were enabled.
- Action links for the workflows teams actually use, such as onboarding, viewing deployment details, or following the preferred release path.
- Lightweight status signals instead of a wall of operational trivia.

This sounds obvious, but it took discipline. Every internal system wants a tile. Every team wants one more field. The useful version of a plugin is the one that says no often enough that the page still makes sense six months later.

### Integration work was the real engineering

The React components were not the hard part. The integration boundaries were.

Backstage is excellent when your source of truth is clear. Internal platforms are rarely that clean. Ownership may live in catalog annotations, deployment state in another service, workflow templates somewhere else, and environment policy in a fourth place. The plugin only feels simple after you do the unglamorous work of deciding which system wins when data conflicts.

We ended up treating the catalog entity as the identity anchor and composing the rest around it. That gave us a stable key for joining related platform data and kept the plugin aligned with how engineers already navigated services in Backstage.

It also forced a healthy rule: if some critical platform fact could not be tied back to a catalog entity cleanly, we treated that as a platform modeling problem, not a UI problem.

### What improved once it shipped

The biggest win was not that the plugin added new capability. It reduced hesitation.

Teams no longer had to remember where a workflow lived or which internal doc explained the "right" path. The plugin put the common actions and the surrounding context on the same page as the service itself. That shrank the distance between "I found my service" and "I know what to do next."

A few improvements showed up quickly:

- Platform onboarding conversations got shorter because the plugin made the happy path visible.
- Service ownership questions got answered faster because the metadata was no longer buried.
- The platform team got fewer "where do I do X?" questions and better feedback on the workflows that were still confusing.
- Backstage felt more like an engineering home page and less like a catalog with some extras attached.

### What I would keep in mind next time

If I had to do the project again, I would keep the same three rules.

**Treat the plugin as a product surface, not a dashboard.** A dashboard accumulates data. A product surface helps someone complete a task.

**Normalize the model before polishing the UI.** A beautiful interface over inconsistent platform data becomes expensive immediately.

**Resist the urge to centralize everything in the plugin.** Backstage works best when it guides, explains, and connects. It does not need to impersonate every downstream system to be valuable.

### Takeaway

Building the Backstage plugin for the TCS Labs internal developer platform was less about adding another internal tool and more about making the platform legible. That is the part people underestimate. Platform engineering is not done when the APIs work. It is done when engineers can understand the platform well enough to use it confidently without needing a tour guide.
