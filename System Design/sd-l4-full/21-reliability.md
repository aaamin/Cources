# Lesson 21 — Reliability, Overload & Failure Isolation

**Phase:** Fundamentals  
**Session:** 21/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn the production patterns that keep slow or failing dependencies from cascading through the entire system.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Timeouts

Every remote call needs a bounded wait. Without a timeout, slow dependencies hold threads, connections, and memory indefinitely. Timeouts should fit an end-to-end latency budget so one dependency cannot consume the whole request deadline.

## Retries, backoff, jitter

Retries help transient failure, but immediate retries multiply load exactly when the dependency is unhealthy. Exponential backoff spreads retries over time; jitter prevents thousands of clients from retrying in lockstep. Retry only safe/idempotent operations.

## Circuit breaker

A circuit breaker stops sending requests to a dependency that is clearly failing. While open, callers return an error or fallback rather than consuming more resources. Periodic probes test recovery.

## Bulkheads

Separate resource pools prevent one dependency/tenant/workload from exhausting everything. Critical password-reset notifications can have separate queues/workers from marketing campaigns, so bulk traffic cannot starve critical messages.

## Backpressure and admission control

When saturated, accepting unlimited work makes latency and memory worse. Backpressure slows upstream producers; admission control rejects excess work before it consumes scarce resources. Controlled rejection is often better than collapse.

## Load shedding and graceful degradation

Under severe pressure, remove optional work and preserve core functionality. A feed might disable expensive recommendations while still showing chronological posts. This deliberately trades quality/features for availability.

## Failure domains

Replicas should span independent failure domains—hosts, zones, or regions depending on the requirement. Three copies on one physical machine do not protect against that machine failing.

## Worked example — slow recommendation dependency

A home page normally calls Recommendations in 50 ms, but it rises to 5 seconds. Set a short timeout, avoid user-path retry storms, open a circuit breaker after sustained failure, fall back to cached/generic recommendations, and isolate its resource pool so checkout/login remain healthy.

## Interview lens

Interviewers often ask “what if it becomes slow?” because slow dependencies can be more dangerous than dead ones: they keep consuming resources while making little progress.

## What to remember

Reliability is failure containment: timeout, retry carefully, isolate, apply backpressure, shed load, and degrade intentionally.

## Review after reading

1. Why can retries amplify outage?
2. What does jitter do?
3. Circuit breaker purpose?
4. What is a bulkhead?
5. Why reject some traffic under overload?

## Deeper study notes

### Build a timeout budget top-down

If the user request SLO is 500 ms, an internal service should not wait 2 seconds for a dependency. Allocate portions of the budget to network, app work, and downstream calls. Nested timeouts should generally get shorter deeper in the call chain.

### Retry only when probability of success improves

Retrying a validation error or permanent permission failure is useless. Retry connection resets, transient 5xx, or throttling only when the operation is safe and the dependency has a chance to recover. Honor `Retry-After` when available.

### Overload looks like failure

A service can be healthy but saturated. Queueing causes latency, timeouts trigger retries, retries add more load, and the system enters a feedback loop. Admission control breaks that loop by limiting active work.

### Graceful degradation should be designed ahead of time

A system cannot invent a safe fallback during an incident. Decide which features are optional, which cached/stale data is acceptable, and which operations must fail closed. This turns outage behavior into an architectural property.

### Common mistakes

- Same timeout for every dependency regardless of SLO.
- Retrying non-idempotent writes automatically.
- Unlimited retry count.
- Circuit breaker without a useful fallback or probe policy.
- Calling load shedding “data loss” when rejected work was never accepted.


## Important interview ideas

> **Important:** Reliability patterns should prevent **cascading failure**. A single slow dependency should degrade one feature, not consume all resources and bring down the whole service.

### Timeout budgets

If an endpoint has a 500 ms p99 target, a downstream timeout of 5 seconds makes no sense. Allocate an end-to-end budget:

```text
Gateway 30ms
Service logic 50ms
DB 150ms
optional dependency 100ms
network/queueing/headroom
```

The exact budget varies, but callers should stop waiting before the user-level deadline.

### Retry budgets

Retries multiply load. If every layer retries 3 times across a 3-service chain, one original request can explode into many attempts.

Prefer retries at one appropriate layer, with exponential backoff/jitter and only for transient/idempotent operations.

### Circuit breaker state

Conceptually:

```text
CLOSED → calls pass
OPEN → fail fast / fallback
HALF-OPEN → limited probe calls
```

A circuit breaker is not a replacement for timeout; it learns that repeated calls are likely wasteful.

### Bulkheads and priority

Separate scarce resources for workloads with different importance. For example:

```text
password reset workers
marketing workers
```

If a marketing provider stalls, reset emails still flow. Bulkheads can be separate thread pools, queues, connection pools, or service instances.

### Backpressure vs load shedding

Backpressure asks upstream to slow down. Load shedding deliberately rejects work when capacity is exhausted. Both are better than accepting unlimited requests into unbounded queues.

### Graceful degradation

Define what can be removed while preserving the core product:

- recommendation service down → generic list;
- analytics down → queue/drop according to policy;
- thumbnail service down → show original/placeholder;
- search down → direct ID lookup still works.

## Worked scenario — recommendation outage

A home feed requires posts, but recommendation ranking is optional. Ranking starts timing out.

Design:

1. 150 ms ranking timeout;
2. circuit opens after sustained errors;
3. serve cached/chronological feed;
4. separate ranking connection pool from core DB pool;
5. alert on failure rate and open circuit;
6. half-open probes after cooldown.

Users get a lower-quality feed rather than an outage.

## Interview questions and model answers

### Q1. “When should I retry?”

Only when the error is plausibly transient and the operation is safe/idempotent. Use bounded attempts with backoff/jitter. Do not retry validation errors or indefinitely retry an overloaded service.

### Q2. “Circuit breaker vs rate limiter?”

A circuit breaker reacts to dependency failure and stops calls temporarily. A rate limiter controls allowed request volume based on policy. They solve different problems and may coexist.

### Q3. “What is backpressure?”

A mechanism that prevents producers from overwhelming consumers by slowing, blocking, or rejecting new work. Queue depth/consumer capacity can drive it.

### Q4. “What is graceful degradation?”

Intentionally reducing non-essential functionality under failure/overload so critical features remain available. The degraded mode should be planned, observable, and safe.

## Common mistakes to avoid

- Retrying at every layer.
- No timeout.
- Retry non-idempotent writes blindly.
- Infinite queues.
- Circuit breaker with no fallback/error behavior.
- One worker pool for critical and bulk work.
- Treating every feature as equally essential.

## Short revision note

Failure containment sequence: **timeout → bounded retry → circuit breaker → isolation/bulkhead → backpressure/load shedding → graceful degradation**.

## Topics to revise

- [ ] timeout budget
- [ ] retry/backoff/jitter
- [ ] retry amplification
- [ ] circuit breaker
- [ ] bulkhead
- [ ] backpressure
- [ ] admission/load shedding
- [ ] graceful degradation

## Interview-ready synthesis

### A strong 60–90 second explanation

I design failure containment, not just retries. Remote calls have bounded timeouts; retries are selective and jittered; repeated dependency failure opens a circuit; resources are isolated with bulkheads; overload triggers backpressure/admission/load shedding; optional features degrade rather than bringing down the core path.

### How this topic connects to the wider system

- Reliability: contains cascading failure and retry amplification.
- Performance: bounded queues/timeouts protect latency under saturation.
- Scalability: overload policy matters when demand exceeds provisioned capacity.
- Product behavior: graceful degradation prioritizes critical user actions.

### Revision flashcards with answers

**Backoff?**  
Increasing delay between retry attempts to reduce repeated pressure.

**Jitter?**  
Randomness that prevents synchronized retry waves.

**Bulkhead?**  
Separate resource pool limiting one workload's blast radius.

**Load shedding?**  
Reject/drop lower-priority work when saturated to preserve core service.

**Circuit breaker?**  
Temporarily fail fast when dependency failure rate indicates calls are likely useless.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Home feed: ranking timeout falls back to chronological feed rather than failing page.
- Notifications: separate critical/bulk worker pools are a bulkhead.
- Cache outage: load shedding protects database from full fallback traffic.

### When not to overuse this idea

Do not add retries before timeouts/idempotency and capacity analysis. Retries can convert a partial incident into a full overload.

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

**Next:** Lesson 22 — Idempotency, Transactions, Concurrency & Distributed Workflows
