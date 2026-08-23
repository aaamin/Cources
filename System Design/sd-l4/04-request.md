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
