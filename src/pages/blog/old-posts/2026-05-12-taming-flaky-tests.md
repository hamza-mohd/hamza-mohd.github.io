---
title: Taming Flaky Tests - A Diagnostic Approach
publishDate: 05-12-2026
author: Hamza Mohd
description: |
    Flaky tests are not random. They are bugs with a probability attached. A practical method for actually fixing them.
keywords: "flaky tests, testing, ci, debugging, reliability, software engineering"
layout: "../../../layouts/BaseLayout.astro"
---

The most expensive bug in many engineering organizations is not in production. It is in the CI pipeline. A flaky test costs every engineer on the team a little bit of time, every day, forever, until someone fixes it. Multiply by a team of fifty and a year of weeks, and you have spent more on retries than you would have on a dedicated infrastructure project.

The reason flaky tests survive is that the easy response - "just retry it" - works locally. The test passes on the second run. The PR merges. The pattern continues. Six months later, the CI signal is so noisy that nobody trusts a failure until they have run it twice, which means the one real regression of the quarter sits in the queue for an extra day before anyone investigates.

Flaky tests are not random. They are bugs with a probability attached. They deserve the same diagnostic discipline as any other bug.

### Categorize before you fix

In my experience, the vast majority of flaky tests fall into five buckets. Naming the bucket tells you the fix.

**1. Order dependence.** The test passes when run alone but fails when run after some other test that mutates shared state (a global, a singleton, a database row, a filesystem path). The fix is almost always to isolate the state, not to fix the symptom. Per-test transactions, per-test temp directories, per-test fixtures. If the test framework supports random ordering, turn it on.

**2. Time dependence.** The test asserts something about "now" or relies on a timeout. It passes on a fast machine and fails on a slow one, or vice versa. The fix is to inject a clock, mock the time, or replace wall-clock waits with explicit synchronization. Anything of the form `time.Sleep(100ms)` followed by an assertion is a future flake.

**3. Concurrency.** The test exercises concurrent code and asserts on a result that depends on scheduling. Sometimes the assertion happens before the goroutine finishes. Sometimes two goroutines race for an output. The fix is explicit synchronization (channels, wait groups, conditions) and assertions that wait for a stable state instead of sampling at an arbitrary moment.

**4. External dependencies.** The test hits a real network service, a real DNS resolver, a real time server. It passes when the dependency is up and fails when it is not. The fix is to stub at the boundary - not deep inside the code under test, but at the layer where it meets the outside world. A flaky test that hits the real internet is not testing your code. It is testing the internet.

**5. Resource pressure.** The test fails only under load: shared CI runners, parallel test execution, low memory. These are the hardest to track down because they do not reproduce on a developer laptop. The fix is usually to reduce the test's resource footprint, but sometimes the failure is real and points at a code-under-test bug that only manifests under contention.

### How to actually diagnose one

When a test flakes, do not retry it. Run it in a loop:

```bash
for i in {1..100}; do go test -run TestThatFlaked ./pkg/... || break; done
```

If it fails within 100 runs, you have a reproducer. From there:

- Run it with `-race` (Go) or the equivalent thread sanitizer for your language. Concurrency flakes often surface as data race warnings on the first failing run.
- Run it with the suite around it. If it only fails when combined with other tests, you have an order dependence.
- Run it under load. `stress` or running the test in parallel with itself often surfaces resource and concurrency issues.
- Add logging that captures the actual state at the point of assertion. Most flaky tests have a moment where "what the test expects" and "what the system produced" diverge, and the divergence is informative.

If it does not fail within 1000 runs and you cannot reproduce, the test is probably not the source of the flake. Look upstream: the test runner, the CI environment, the network.

### The hard part is the process

Tooling is easy. The hard part is the team agreement that flaky tests get fixed, not skipped.

A pattern that has worked: every flaky test gets one of three outcomes within a fixed window (a week is a reasonable default).

1. **Diagnosed and fixed.** Default outcome. The work is small if you catch it early.
2. **Diagnosed and quarantined.** If the underlying bug is real but the fix is large, the test is moved to a quarantine suite that runs but does not block. A ticket is filed with the diagnosis. Quarantine has a TTL, not an open-ended grace period.
3. **Deleted.** If the test is not testing anything important, or the cost of maintaining it exceeds its value, delete it. Tests are code. Code that costs more than it pays gets removed.

The outcome that is not on the list is "retry it and move on." Once retries become the default, every test slowly degrades, because nobody is on the hook for any single one.

### A small note on retries

There is a narrow case where automatic retries make sense: tests that talk to systems with known low-rate transient failures (a real cloud API, a real DNS lookup) where the failure mode is well-understood. Even there, the retry should be in the test, scoped to the specific operation that can transiently fail, with a comment explaining why. A blanket "retry the whole suite on any failure" rule turns CI into a slot machine.

### The point

Flaky tests are a slow tax on the entire team. They feel cheaper than they are because the cost is distributed. They are also tractable: name the bucket, build a reproducer, fix the root cause, and resist the temptation to retry. A green CI signal that actually means something is one of the highest-leverage things you can give a team.
