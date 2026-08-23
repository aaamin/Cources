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
