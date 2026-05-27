---
title: When Channels Hurt - Rethinking Go Concurrency
publishDate: 11-03-2025
author: Hamza Mohd
description: |
    Channels are not the default answer in Go. A look at when a mutex, a sync primitive, or a single goroutine is the better tool.
keywords: "golang, go, concurrency, channels, mutex, sync, goroutines"
layout: "../../../layouts/BaseLayout.astro"
---

A lot of Go code I review uses channels where a mutex would be clearer, smaller, and faster. The pattern is so common that I have started to think of it as the Go equivalent of "we needed to pass data, so we used a microservice."

Channels are a beautiful primitive. They are also one of the easier ways to write code that deadlocks, leaks goroutines, or hides a data race behind a layer of misdirection. The Go proverb "do not communicate by sharing memory; share memory by communicating" is good advice when the *communication* is the actual problem you are solving. It is bad advice when you just need to protect a counter.

### A simple example

Here is a cache that I see versions of constantly:

```go
type cache struct {
    set chan setOp
    get chan getOp
}

type setOp struct{ k, v string }
type getOp struct {
    k    string
    resp chan string
}

func (c *cache) run() {
    m := map[string]string{}
    for {
        select {
        case op := <-c.set:
            m[op.k] = op.v
        case op := <-c.get:
            op.resp <- m[op.k]
        }
    }
}
```

This works. It also costs a goroutine, two channels, an allocation per read (the response channel), and a lifecycle problem (when does `run` exit?). The version with a mutex is shorter, faster, and has no lifecycle:

```go
type cache struct {
    mu sync.RWMutex
    m  map[string]string
}

func (c *cache) Get(k string) string {
    c.mu.RLock()
    defer c.mu.RUnlock()
    return c.m[k]
}

func (c *cache) Set(k, v string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.m[k] = v
}
```

The mutex version is also easier to reason about under failure. There is no goroutine to leak, no buffered channel to fill, no select to starve.

### A rule of thumb

I have settled on a simple heuristic for which primitive to reach for:

- **Protecting shared state in place?** Use `sync.Mutex` or `sync.RWMutex`.
- **One-shot computation that returns a value?** Use `sync.Once`, or a function that returns directly.
- **Waiting for N things to finish?** Use `sync.WaitGroup` or `errgroup.Group`.
- **Passing values between independent stages of work?** Now use a channel.
- **Coordinating cancellation?** Use `context.Context`.

The channel case is narrower than it looks. A pipeline of producers and consumers, a fan-out across workers, a select over multiple event sources - those are real channel use cases. A counter, a cache, a flag, or a "tell me when you are done" handshake usually is not.

### Subtleties that bite

Even when channels are the right tool, a few patterns reliably cause problems:

**Unbuffered sends in a request path.** An unbuffered channel send blocks until a receiver is ready. If the receiver has gone away (panicked, context-cancelled), the sender hangs forever. Wrap sends in a `select` with a context or use a buffered channel with a documented drop policy.

**Goroutines without owners.** Every goroutine you start needs an answer to "who shuts this down, and how?" If the answer is "it runs until the program exits," that is a leak waiting to happen the first time the code is used in a test or a long-running process.

**Channels as event buses.** Once a channel is used by more than two goroutines, the rules about who closes it become folklore. `close` panics on send, panics on double-close, and returns the zero value on receive. A single-writer, multi-reader channel is fine. A multi-writer channel needs a coordinator (often a `sync.WaitGroup` plus a separate goroutine that closes once everyone is done).

**`select` with a `default`.** A non-blocking select feels like a free lunch and is almost never what you want. It turns "wait for an event" into "spin-poll for an event," which is a CPU bug dressed up as concurrency.

### When channels really do shine

I do not want to leave the impression that channels are bad. They are the right answer for:

- Pipelines where each stage has clear ownership and a clear close signal.
- Bounded worker pools where the channel acts as the work queue and as the backpressure mechanism.
- Combining multiple cancellation or event sources in one place (`select` over `ctx.Done()`, a timer, and an inbound message).

The common thread is that channels are about *flow*. When the problem is "things flow from here to there," they are the cleanest primitive in any mainstream language. When the problem is "two callers must not stomp on the same map," they are the wrong tool wearing a Go costume.

### The point

Reach for the smallest primitive that fits the problem. A mutex around a map is not a failure of Go style. It is good Go style. The point of the channel primitive is that you *can* use it for coordination - not that you must use it for every coordination.
