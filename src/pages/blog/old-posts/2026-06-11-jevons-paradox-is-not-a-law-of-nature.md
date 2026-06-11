---
title: Jevons Paradox Is Not a Law of Nature
publishDate: 06-11-2026
author: Hamza Mohd
description: |
    Efficiency sometimes increases consumption, but not always. Jevons' paradox is a conditionally useful idea, not an automatic rule.
keywords: "jevons paradox, efficiency, economics, energy, ai, productivity, systems thinking"
layout: "../../../layouts/BaseLayout.astro"
---

Jevons' paradox has become one of those ideas that people reach for whenever a technology gets more efficient: if the cost of doing something falls, demand will rise, and total consumption may go up instead of down.

Sometimes that is exactly what happens. But the way the idea is invoked now often turns it into something much stronger than it is. It gets used like a law of physics: *efficiency improvements will always backfire because people just use more.*

That is too strong. Jevons' paradox is real in some settings, but it is not a universal rule. It depends on demand elasticity, substitution effects, budgets, bottlenecks, and whether there is actually more useful work waiting on the other side of lower cost.

In other words: **efficiency lowers cost. It does not guarantee infinite appetite.**

### What Jevons actually gets right

The core intuition is sound.

If a resource becomes dramatically cheaper to use, people often find more uses for it. Steam engine improvements made coal-powered work cheaper, which helped expand the industrial uses of coal rather than shrink them. Similar rebound effects show up elsewhere:

- cheaper compute enables workloads that were previously uneconomical
- better network capacity encourages richer applications
- faster databases invite more product features to depend on data access

Engineers see versions of this all the time. Cut the cost of an API call enough and someone will add five new screens that depend on it.

The mistake is moving from **"rebound effects exist"** to **"efficiency gains must increase aggregate consumption."**

That second claim needs more assumptions than people usually acknowledge.

### Demand is not infinitely elastic

The first constraint is obvious once stated plainly: people do not want an infinite quantity of most things.

If I make a build pipeline 40% faster, engineers will appreciate it. They will not suddenly run thousands of builds per hour just because they can. If I make a note-taking app 10x cheaper to host, users are not going to generate 10x as many meaningful notes overnight. If disk space gets cheaper, people store more, but not without some upper bound imposed by actual use.

For Jevons-style expansion to dominate, the newly cheaper thing has to sit on a demand curve with enough room to move. Some technologies do. Many do not.

Efficiency only turns into much larger total consumption when lower cost unlocks a lot of latent demand that was truly waiting there.

### Other constraints often dominate

A second reason the paradox is not universal is that the efficient resource is often not the real bottleneck.

You can make inference cheaper, but maybe product review capacity is the bottleneck. You can make software development faster, but maybe customer acquisition is the bottleneck. You can make cloud storage cheaper, but maybe the scarce resource is not storage - it is human attention, organizational coordination, or willingness to pay.

Systems are full of coupled constraints. Lowering one input cost matters, but it does not automatically expand the whole system if another limit binds first.

That is why organizations often overestimate the downstream impact of technical efficiency gains. They model the system as if one cheaper component releases all demand. Usually it just moves the pressure somewhere else.

### Sometimes efficiency is captured as savings

Another missing assumption is that every efficiency gain gets reinvested into more throughput.

Sometimes it does. Sometimes the gain is simply harvested.

If a company cuts infrastructure cost by 30%, it might:

1. run larger workloads
2. offer a cheaper product and expand demand
3. keep the same workload and improve margins

Option three is extremely common. Jevons-style reasoning often ignores it because it is less dramatic, but financially it is often the default behavior.

The same is true in personal productivity. If a task takes half as long, people do not always fill the reclaimed time with twice as much output. Sometimes they spend it on higher-quality work, waiting less, context-switching less, or just having slack. That is not paradox. That is a different objective function.

### The AI version is especially overstated

You can see the overreach clearly in current discussions about AI and software.

The strong claim goes like this: if models make content, code, or analysis cheaper, total production will explode, therefore overall compute use will inevitably keep rising, therefore efficiency gains do not really save anything.

Maybe. But "inevitably" is doing too much work.

A few different futures are possible:

- some teams may use cheaper intelligence to do much more work
- some may do the same work faster and stop there
- some may raise quality expectations rather than volume
- some use cases may saturate quickly because the scarce resource is trust, review, distribution, or taste

More efficient models clearly lower the cost floor. What happens to total usage after that is an empirical question, not a theorem.

### Rebound is strongest when three conditions hold

Jevons-like outcomes are most plausible when:

1. the efficiency gain is large enough to matter materially
2. there is a lot of unmet demand waiting behind the old cost barrier
3. no stronger constraint steps in before usage expands

That is a useful frame because it turns the conversation from slogan to diagnosis.

When someone says, "this efficiency improvement will just cause more usage," the right follow-up questions are:

- More usage by whom?
- For which marginal use cases?
- What budget or coordination constraint disappears?
- Where is the latent demand coming from?
- What prevents the gain from being taken as savings instead?

If those questions do not have answers, the claim is probably too hand-wavy to trust.

### Why the stronger claim persists

Part of the appeal is rhetorical. "Efficiency makes the problem worse" is a memorable line. It has the flavor of counterintuitive wisdom, which makes it travel well.

It is also directionally true often enough to feel profound. Engineers have all seen cases where performance wins invited more features and the savings disappeared. Once you have lived through that cycle, it is easy to over-generalize from it.

But good systems thinking is mostly the discipline of refusing to universalize a local pattern too quickly.

Jevons' paradox is best treated as a possibility you check for, not a conclusion you import by default.

### The practical takeaway

If you are evaluating an efficiency gain - in energy, software, AI, or infrastructure - ask two separate questions:

**First:** how much does the unit cost fall?

**Second:** what does the surrounding system do with the saved capacity?

Those are different questions. The first is engineering. The second is economics, product strategy, and organizational behavior.

Sometimes the answer will be "usage explodes." Sometimes it will be "nothing much changes." Sometimes it will be "the system gets saner because the same work now costs less."

That last outcome is so ordinary that people forget to argue for it. They should not.

Efficiency is not meaningless because rebound exists. And Jevons' paradox is not false because it is conditional. It is just not a law of nature.
