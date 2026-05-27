---
title: The Indexes You Don't See
publishDate: 01-19-2026
author: Hamza Mohd
description: |
    A field guide to the database indexing mistakes that only show up under production-shaped load.
keywords: "database, postgres, mysql, indexing, performance, query planning, sql"
layout: "../../../layouts/BaseLayout.astro"
---

The query is fast in development. It passes review. It ships. Three months later, a dashboard goes from "snappy" to "spinner" and someone is paged at 2am to figure out why.

The cause is almost always an index that was missing, redundant, or the wrong shape for the query that actually runs in production. Indexing is one of the parts of our craft where the gap between "what looks right" and "what works" is widest. A few patterns come up often enough that they deserve to be named.

### The composite index direction matters

`CREATE INDEX ON orders (customer_id, created_at)` is not the same as `CREATE INDEX ON orders (created_at, customer_id)`. The first supports queries like "all orders for this customer, newest first." The second supports queries like "all orders in this date range, optionally filtered by customer."

The rule: the leading columns of a composite index must match the equality predicates of your query. Range predicates and ordering can use the trailing columns. Get the order wrong and the planner falls back to a sequential scan or, worse, a bitmap heap scan that looks fine in `EXPLAIN` and dies under concurrency.

### The "small table" trap

A table with ten thousand rows seems small enough that you do not need an index. In a single-query benchmark, it is. In production, that table is read on every request, joined against three other tables, and the planner is making a different choice every time depending on statistics. The query that took 2ms in isolation takes 200ms under load because every replica is racing the same sequential scan.

If a table is read on the hot path, it gets an index on the predicate columns. Size is not an excuse.

### Indexes that exist but are not used

This is the most demoralizing one. You add an index. The query is still slow. `EXPLAIN` shows a sequential scan. The index is right there.

Common causes:

- **Type mismatch.** `WHERE user_id = '123'` against a `bigint` column will not use the index because the planner casts the column, not the literal. Quote your literals to match the column type, or fix the cast at the application layer.
- **Function on the indexed column.** `WHERE lower(email) = 'me@example.com'` will not use an index on `email`. Either index the expression (`CREATE INDEX ON users (lower(email))`) or normalize at write time.
- **`OR` across unindexed columns.** `WHERE a = 1 OR b = 2` will skip the index on `a` unless `b` is also indexed. Often a `UNION ALL` of two indexed queries is faster.
- **Selectivity below the threshold.** If 40% of the rows match the predicate, the planner is right to skip the index. A sequential scan with a filter is genuinely faster than 40% of a B-tree.

When the index is ignored, the question is "why does the planner think the scan is cheaper," not "how do I force the index." Forcing the index hides the underlying mismatch.

### Redundant indexes are not free

A B-tree index has to be updated on every write. If you have indexes on `(a)`, `(a, b)`, and `(a, b, c)`, the first two are redundant: the third covers them as prefixes. You are paying for three writes when one would do, and you are using three times the buffer cache.

`pg_stat_user_indexes` in Postgres (or equivalent in your engine) shows which indexes are actually getting scanned. An index with zero scans after a month of production traffic is dead weight. Drop it and the write path gets faster for free.

### Covering indexes are underused

A "covering" index includes every column the query needs, so the engine can answer the query from the index alone without touching the heap. Postgres supports this explicitly with `INCLUDE`:

```sql
CREATE INDEX ON orders (customer_id, created_at) INCLUDE (status, total);
```

For a hot read query, this can turn three I/Os into one. It is not a free lunch (the index gets larger, writes get slower), but for read-heavy tables it is one of the highest-leverage optimizations available.

### Partial indexes for the long tail

A lot of "always filter by `status = 'active'`" queries are perfect candidates for partial indexes:

```sql
CREATE INDEX ON jobs (created_at) WHERE status = 'active';
```

The index only contains rows that match the predicate, which is usually a small fraction of the table. Reads against active jobs are fast. Writes to completed jobs do not touch the index at all. The downside is that the planner has to be able to prove that your query matches the partial predicate, which means you need to literally include `WHERE status = 'active'` in the query.

### Production-shape testing

The best defense against index regressions is to test against production-shaped data. That does not mean a full copy of production. It means:

- A staging dataset with realistic *cardinality* and *distribution*. A table with 100 rows where 80 share the same `tenant_id` does not behave like a table with 100 million rows where the same is true.
- A `EXPLAIN (ANALYZE, BUFFERS)` step in code review for any new query on a hot table. The plan in review is the plan you will get in production, modulo statistics.
- A small library of "slow queries we have seen" with reproducers. Every incident contributes one. Future regressions get caught by the same harness that caught the original.

### The point

Indexes are not a deployment-time decision. They are part of the query design. Every read on a hot path has an index that supports it; every index that exists is justified by a query that uses it. Anything else is overhead waiting to find you.
