# Lesson 04 — Client–Server Architecture & Request Lifecycle

**Phase:** Fundamentals  
**Session:** 4/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn how requests move through a backend, how stateful and stateless services differ, where latency accumulates, and why long synchronous call chains are fragile.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Client and server

The client initiates operations; servers authenticate, validate, apply business rules, access state, and coordinate dependencies. Even `Client → Server → Database` contains important questions about identity, errors, transactions, resource limits, and retries.

## Stateless services

A stateless application server does not require the same client to return to the same process for correctness. Durable/shared state lives in databases, caches, or dedicated stores. Any healthy instance can serve the next request, making load balancing and failover easier.

## Stateful services

Some services naturally own live state: WebSocket gateways, game sessions, stream processors. Stateful is not wrong; it means routing, recovery, ownership, and failover require explicit design. A system may need to know which gateway currently owns a user's connection.

## Synchronous call chains

A chain such as `API → Order → Inventory → Payment → DB` accumulates latency and failure probability. One slow dependency makes the whole path slow. Non-critical work such as email should often move to asynchronous processing.

## Connection pools

Servers reuse finite pools of database or HTTP connections. If dependencies slow down, pools fill, requests wait, and latency rises. This is an early example of backpressure and why downstream capacity limits upstream throughput.

## Worked example — checkout request lifecycle

A checkout request can synchronously validate inventory, charge payment, and persist the order because those steps determine the user-visible result. Confirmation email can be `Checkout → queue → notification worker`, so an email-provider slowdown does not block checkout.

## Interview lens

When narrating a request, say which steps are synchronous, where durable state changes, and which work can happen later. Interviewers often probe the weakest synchronous dependency.

## What to remember

Think about state ownership, latency, finite resource pools, synchronous dependencies, and failure propagation—not just arrows.

## Review after reading

1. Why are stateless servers easy to scale?
2. Give a naturally stateful service.
3. Why are long synchronous chains risky?
4. What happens when a connection pool fills?
5. Which checkout tasks can be async?

## Deeper study notes

### Follow one request, then follow one failure

A strong way to understand an architecture is to narrate one success path and then repeat the path with one dependency failing. For example, in checkout: client → API → inventory → payment → order DB. Now make payment slow. Which timeout fires? Does inventory remain held? Can the client retry? This turns a static diagram into a system.

### State has several forms

“Stateful” is broader than database rows. A server may own connection state, in-flight workflow state, local cache, session state, file handles, or leases. Ask which state is durable and which can be reconstructed. Stateless app servers are easier to replace precisely because local state is not required for correctness.

### Synchronous work belongs on the critical path only when required

A user should wait for work that determines the correctness of the requested operation. They usually should not wait for analytics, email, thumbnails, or search indexing. Moving optional work behind a durable event/queue reduces tail latency and isolates external providers.

### Fan-out increases tail risk

If one request calls five services in parallel and needs all five responses, the slowest dependency controls completion. If it calls them sequentially, latencies add. This is why distributed systems care about p95/p99 latency rather than only averages.

### Resource queues exist even when you do not draw them

Requests wait for CPU, event-loop turns, thread pools, database connections, locks, and downstream capacity. As utilization approaches saturation, queueing delay rises sharply. “Just add retries” can make this much worse.

### Common mistakes

- Calling every downstream dependency synchronously.
- Storing user session only in one app instance without considering failover.
- Ignoring what happens after the caller times out.
- Treating the database connection pool as infinite.
- Drawing services but not narrating read/write flows.


## Important interview ideas

> **Important:** A system diagram is useful only if you can narrate a request through it. For every important user action, know the synchronous path, durable state changes, and asynchronous side effects.

### The critical path

The **critical path** is the sequence of work that must complete before the user receives the required response. Keep it small.

For checkout, the critical path may include inventory reservation and payment authorization. Sending email, analytics, recommendation refresh, and search indexing usually do not belong there.

A useful question is:

> “If this step takes five seconds, does the user truly need to wait for it?”

If no, consider durable asynchronous processing.

### State classification

When examining a server, classify state into three groups:

1. **Durable business state** — orders, payments, messages. Must survive process failure.
2. **Reconstructable shared state** — cache entries, search indexes, feed materializations.
3. **Ephemeral process state** — active connections, in-flight request buffers, temporary local cache.

This classification helps decide whether a server can be stateless and how failure recovery works.

### Tail latency in dependency graphs

If a request synchronously depends on multiple services, user latency depends on their combined behavior. Sequential calls add latency. Parallel calls are controlled by the slowest required dependency. At p99, one flaky service can dominate the entire endpoint.

This is why optional data often gets a fallback. The page may return without recommendations if the recommendation service is slow.

### Resource exhaustion is often the actual outage

A dependency slowing down can exhaust:

- request threads/tasks;
- database connection pools;
- HTTP client pools;
- memory holding request state;
- queue capacity.

The upstream service may then fail even though its CPU is low. This motivates timeouts, bulkheads, backpressure, and bounded queues.

## Worked scenario — checkout critical path

A first design does this synchronously:

```text
Checkout
  → Inventory
  → Payment
  → Email Provider
  → Analytics
  → Recommendation Engine
  → save order
```

This is fragile. Email or analytics failure can block purchase.

A better decomposition is:

```text
Checkout
  → reserve inventory
  → authorize payment
  → persist confirmed order + outbox
  → return success

outbox/event
  ├→ email
  ├→ analytics
  └→ recommendations
```

Now non-critical side effects can retry independently.

## Interview questions and model answers

### Q1. “What makes a service stateless?”

Correctness does not depend on a particular process retaining client-specific state between requests. Durable/shared state lives in a database, cache, or other service. An instance can die and another can handle the next request. Local caches may still exist; stateless means local state is not required for correctness.

### Q2. “Are WebSocket servers stateless?”

They hold connection state, so operationally they are stateful. But business data can still be external. If a gateway dies, the client reconnects elsewhere and reconstructs session state from durable data. This makes the state ephemeral rather than irreplaceable.

### Q3. “When should I introduce a queue?”

When work does not have to complete on the user path, is bursty, depends on a slow external provider, or benefits from independent retries. I would not add a queue for a tiny synchronous read that requires an immediate answer.

### Q4. “Why can a slow dependency be worse than a down dependency?”

A down dependency often fails fast. A slow dependency keeps requests and connections occupied, so resources accumulate until upstream services saturate. Timeouts and circuit breakers contain this.

## Common mistakes to avoid

- Making optional work synchronous.
- Treating process-local session state as durable.
- Not knowing where the source of truth lives.
- Ignoring connection-pool limits.
- Drawing arrows without describing failure behavior.
- Assuming a timeout cancels remote work.
- Allowing one slow dependency to consume every worker/thread.

## Short revision note

For every request, identify **critical path → durable writes → async side effects → failure points → resource limits**.

## Topics to revise

- [ ] stateless vs stateful
- [ ] durable vs ephemeral state
- [ ] synchronous call chain
- [ ] critical path
- [ ] connection pools
- [ ] tail latency
- [ ] async side effects
- [ ] failure propagation

## Interview-ready synthesis

### A strong 60–90 second explanation

For every user action I narrate the critical path: request entry, authorization, required business decisions, durable writes, response, and asynchronous side effects. I classify state as durable, reconstructable, or ephemeral. If a dependency is optional, I try not to make user success wait for it.

### How this topic connects to the wider system

- Performance: synchronous dependency chains add or amplify latency.
- Reliability: slow dependencies can exhaust pools even before they fully fail.
- Scalability: stateless compute is easier to add/remove behind a load balancer.
- Correctness: durable state must be written before acknowledgements when the product promises persistence.

### Revision flashcards with answers

**What is critical path?**  
Work that must finish before the requested operation can truthfully return success.

**What is ephemeral state?**  
State such as an active socket that can be reconstructed after process failure.

**Why make email async after checkout?**  
Email is not required to decide whether the purchase succeeded and external provider latency should not block checkout.

**Why do connection pools matter?**  
They are finite; slow downstream calls can exhaust them and cascade failure.

**Stateless means no local cache?**  
No. Local cache is fine if correctness does not depend on it surviving or staying on one instance.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Chat: connection gateway state is ephemeral, while messages must be durable; reconnect rebuilds connection state from stored history.
- Checkout: payment and inventory belong on the critical path, while email/analytics should be asynchronous.
- Feed: ranking can be an optional synchronous dependency with chronological fallback so one model outage does not take down reading.

### When not to overuse this idea

Do not decompose a small, fast local request into multiple services merely to look distributed. Each remote boundary adds latency, failure, and operational cost.

### A good interviewer sentence

> “I would use this only because the current requirement/workload creates the specific problem it solves. If that assumption changes, I would simplify or choose the alternative.”

This sentence captures an important L4 behavior: architecture is conditional, not dogmatic.

## Personal notes

```text
Concepts that are clear:

Concepts to revisit:

Three things to remember:
1.
2.
3.

Questions for later:
```

---

**Next:** Lesson 05 — Load Balancing
