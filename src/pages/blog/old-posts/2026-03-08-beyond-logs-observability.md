---
title: Beyond Logs - A Pragmatic Take on Observability
publishDate: 03-08-2026
author: Hamza Mohd
description: |
    Logs, metrics, and traces solve different problems. A practical guide to which to reach for, and what to instrument first.
keywords: "observability, opentelemetry, logs, metrics, traces, sre, debugging"
layout: "../../../layouts/BaseLayout.astro"
---

"We have logs" is the most common answer when I ask a team how they debug production. It is also, almost always, the reason their incidents take longer than they should.

Logs are necessary. They are not sufficient. The three signals - logs, metrics, traces - solve different problems, and using one in place of another is a recipe for slow debugging and bloated bills.

### What each signal is good for

**Metrics** answer "how is the system behaving in aggregate, right now and over time?" They are cheap to store, cheap to query, and easy to alert on. They are the signal you watch when you do not know whether anything is wrong.

**Traces** answer "for this one request, what happened?" They are the signal you reach for once you know something is wrong and you need to know *where*. A good trace turns "the checkout endpoint is slow" into "the checkout endpoint spends 400ms waiting on the fraud-check service, which spends 380ms in a single Postgres query."

**Logs** answer "what specifically happened at this point in the code?" They carry context that does not fit in a metric label or a span attribute - the actual values, the error chain, the decision the code made. They are the signal you read once you know which line of code you care about.

The mistake is using logs for all three. Counting how often something happens by grepping logs is a metric pretending to be text. Reconstructing a request path by correlating timestamps across services is a trace pretending to be text. Both work. Both are slow, expensive, and lossy.

### What to instrument first

If you are starting from "we have logs," the highest-leverage additions are, in order:

1. **The RED metrics for every service.** Rate, errors, duration. One histogram and one counter per endpoint. This alone will tell you within seconds whether the problem is "everything is slow," "one endpoint is slow," or "one endpoint is erroring." You do not need a dashboard yet. You need the data.

2. **Request-scoped trace IDs propagated end to end.** Generate a trace ID at the edge, stamp it onto every log line in the request's path, propagate it across service boundaries via headers. Even without a tracing backend, this turns "find all logs related to this request" from a 20-minute exercise into a single grep.

3. **A sampled tracing backend.** Once trace IDs exist, point them at OpenTelemetry plus any compatible backend. Sample aggressively - 1% of normal traffic is usually enough, and you can sample 100% of errors. The point is not to keep every span. The point is to keep enough of them to debug typical and pathological cases.

4. **Structured logs with consistent fields.** `level`, `ts`, `service`, `trace_id`, `span_id`, `error`, plus whatever is specific to the operation. JSON or logfmt, pick one and never mix them. The single biggest unforced error in logging is letting each service invent its own format.

After those four, the marginal value of each new piece of instrumentation drops sharply. A team that has these four and uses them well will out-debug a team that has a wall of fancy dashboards and no trace IDs.

### Cardinality is the silent killer

Every metrics system has the same failure mode: you add a label that takes on too many values and your storage costs explode. The classic offenders are `user_id`, `request_id`, `path` (when the path includes IDs), and `error_message` (when the message includes dynamic content).

A few rules that have held up:

- **No unbounded label values.** If a label can take on more than a few hundred distinct values across the fleet, it is the wrong shape for a metric. It belongs in a trace or a log.
- **Bucket aggressively.** "Request latency by p50/p95/p99" is a metric. "Request latency by exact path string" is a query you should run against traces.
- **Be ruthless in review.** Cardinality bugs are almost always introduced one PR at a time, by someone who did not realize the field they were adding as a label was effectively unbounded.

### Logs are not free either

The same restraint applies to logs. A service that logs at INFO for every request, with the full request body, will spend more on log storage than on compute. A few principles:

- **Log decisions, not data.** "Refused request because tenant is over quota" is useful. "Received request {full JSON dump}" is not, unless you are actively debugging.
- **Sample debug logs in production.** A request that hits a hot path a million times a day does not need a million debug lines. Sample, or gate behind a tenant or feature flag.
- **Treat log volume as a budget.** Each service has a target bytes/second. If a new feature blows the budget, the feature is wrong, not the budget.

### Where alerting fits

Metrics drive alerts. Not logs. Not traces. An alert on "an error log appeared" is the path to alert fatigue, because errors of varying severity all look the same in the log stream. An alert on "the error rate for service X exceeded 1% for 5 minutes" is actionable.

The rule of thumb: every alert is tied to an SLO or a known-bad condition, and every alert has a runbook. Anything else is noise that will eventually be ignored, including by you at 2am.

### The point

Logs, metrics, traces - three signals, three jobs. Use each for what it is good at, instrument the small set of things that pay back the most, and treat cardinality and log volume as the budgets they are. Most teams do not need more telemetry. They need the telemetry they already have to be the right shape.
