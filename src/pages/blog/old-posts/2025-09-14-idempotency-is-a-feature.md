---
title: Idempotency is a Feature, Not an Afterthought
publishDate: 09-14-2025
author: Hamza Mohd
description: |
    Why idempotency keys belong in the API contract from day one, and how to design them so retries are safe.
keywords: "idempotency, api design, distributed systems, retries, http, post"
layout: "../../../layouts/BaseLayout.astro"
---

Every team I have worked with eventually hits the same incident. A client times out, retries, and a customer ends up with two charges, two emails, or two of whatever the endpoint creates. The post-mortem proposes "we should add idempotency keys." The work gets scoped, deprioritized, and then the next incident happens.

Idempotency is not a retrofit. It is part of the contract. Treating it as a feature you ship later means every consumer downstream of your API is silently coupled to the assumption that retries are unsafe, and that assumption leaks into queues, jobs, and human runbooks.

### What idempotency actually means in an API

The HTTP spec defines idempotency in terms of side effects: making the same request `N` times has the same observable effect as making it once. `GET`, `PUT`, and `DELETE` are idempotent by definition. `POST` is not, and that is where most of the pain lives.

Making `POST` safely retriable requires the server to recognize "I have seen this request before" and return the original result instead of doing the work a second time. That recognition needs:

1. A key that the client controls and can repeat across retries.
2. A server-side record of (key -> result) with a defined lifetime.
3. A rule for what happens when the same key arrives with a different body.

Skip any of these and the behavior is a footgun.

### Designing the key

The convention that has held up best is an `Idempotency-Key` header carrying an opaque, client-generated value (typically a UUIDv4 or v7). Two notes from experience:

- **Generate the key once per logical operation, not per retry attempt.** This sounds obvious until you find an SDK that regenerates the key inside its retry loop. The result is "idempotent" requests that are not idempotent at all.
- **Scope the key to the authenticated principal.** Two different tenants sending the same key by coincidence must not collide. The storage key on the server is `(tenant_id, idempotency_key)`, never just the header value.

### Storing the result

The server-side record needs three fields: the key, a fingerprint of the request body, and the response (status code plus body) of the first successful processing. On a repeat request:

- Same key, same fingerprint -> return the stored response.
- Same key, different fingerprint -> reject with `409 Conflict`. The client is reusing a key for a different operation, which is a bug they want to know about.
- Same key, in-flight -> return `409` or block briefly. The right choice depends on whether your clients prefer "fail fast" or "the second call wins."

Lifetime is the part most designs get wrong. Twenty-four hours is too short for some workflows (batch retries the next day) and too long for others (the key table grows unbounded). Pick a value tied to your retry policy, document it in the API reference, and make the TTL visible in the response (`Idempotency-Replayed: true`, or similar).

### What about the underlying write?

The idempotency layer protects the API surface. It does not, by itself, protect the database. Two patterns work well together:

- **Single-shot writes** with a uniqueness constraint that encodes the business invariant (one charge per `(customer_id, external_ref)`, one user per `email`). The constraint is the last line of defense if the idempotency layer fails, the cache is empty, or a buggy client bypasses the header.
- **Outbox pattern for downstream effects.** If the `POST` triggers an email, a webhook, or a queue message, those should be derived from the committed row, not produced inline. Otherwise a retry that hits the cache will skip the side effect, and a retry that misses the cache will duplicate it.

### Common mistakes worth calling out

- **Returning a fresh response on replay.** If the second call returns the *current* state of the resource instead of the *original* response, you have leaked a race condition into the client. They will see a different result depending on whether they retried or not.
- **Caching only the success path.** If the first call fails with a 5xx, the second call has nothing to replay. Some teams cache the failure too (so the client can distinguish "you already tried this and it broke" from "this is a new failure"). Pick a policy and document it.
- **Treating `PUT` as automatically safe.** `PUT` is idempotent in terms of state, but the side effects can still fire on each call. If `PUT /users/123` sends a welcome email on creation, you need the same idempotency logic.

### Where this leaves you

If your API has any `POST` that creates a resource, sends money, or triggers an external effect, treat idempotency as part of v1 of that endpoint. The cost is a header, a small table, and a few branches in the handler. The cost of adding it after the fact is every consumer's retry logic and a handful of customer-visible incidents.

It is one of the highest-leverage things you can put in an API contract. Worth doing once, properly, up front.
