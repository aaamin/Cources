# Session 21 — Reliability, Overload & Failure Isolation

## Outcome

You should be able to design for slow dependencies and overload using timeouts, retries, exponential backoff, jitter, circuit breakers, bulkheads, backpressure, admission control, load shedding, graceful degradation, health checks, and failure-domain thinking.

## The Most Dangerous Failure Is Often “Slow”

A dependency that is fully down may fail fast.

A dependency that takes 30 seconds per request can consume:
- threads/tasks;
- sockets;
- connection pools;
- memory;
- request queues.

Then your healthy service becomes unhealthy.

```text
Dependency slows
    ↓
in-flight requests grow
    ↓
resources exhausted
    ↓
service slows
    ↓
clients retry
    ↓
load grows further
```

This is a cascading failure.

## Timeouts

Every remote dependency should have bounded waiting.

Choose timeout based on:
- end-to-end latency budget;
- downstream normal p99;
- whether retry is allowed;
- user expectation.

Example:
If API target p99 is 1s, giving a downstream call a 5s timeout is incompatible.

### Timeout layers

```text
Client timeout
Gateway timeout
Service timeout
DB command timeout
External API timeout
```

Ideally inner timeouts fit within outer deadlines.

## Retries

Retry only when:
- failure is plausibly transient;
- operation is safe/idempotent;
- there is remaining time budget.

Do not retry:
- validation errors;
- unauthorized requests;
- many permanent failures;
- non-idempotent side effects without protection.

## Exponential Backoff

Instead of:

```text
retry after 10ms
retry after 10ms
retry after 10ms
```

use:

```text
100ms
200ms
400ms
...
```

This reduces pressure on recovering dependency.

## Jitter

If 10k clients all retry at exactly 1s, they create a synchronized retry wave.

Add randomness:

```text
retry around 1s ± random
```

This spreads load.

## Retry Storm

Suppose:
- service receives 10k RPS;
- downstream fails;
- each request retries 3 times.

Downstream may see up to roughly 40k attempts/s.

Retries can turn partial outage into total outage.

Limit:
- attempts;
- concurrency;
- total deadline;
- retry budget.

## Circuit Breaker

Mental state machine:

```text
CLOSED
 normal calls
 failures exceed threshold
    ↓
OPEN
 fail fast / fallback
 cooldown
    ↓
HALF-OPEN
 allow small probes
 success → CLOSED
 failure → OPEN
```

Purpose:
- stop repeatedly calling a failing/slow dependency;
- let it recover;
- protect caller resources.

Circuit breaker is not a replacement for timeout. You need both.

### When not to use blindly

If failures are random per request and dependency is generally healthy, aggressive breaker can unnecessarily block valid calls.

Tune based on service behavior.

## Bulkheads

Separate resources so one workload cannot consume everything.

Examples:
- separate thread/connection pools;
- separate queues;
- reserved worker capacity;
- tenant quotas.

```text
Marketing email pool
Password reset pool
```

A marketing outage should not starve password resets.

The term comes from ship compartments preventing one leak from sinking the whole ship.

## Backpressure

Downstream communicates or system enforces that upstream must slow down.

Examples:
- bounded queue;
- stream consumer pull rate;
- reject new async jobs;
- reduce producer rate;
- flow-control window.

Without backpressure, memory/queue depth grows until failure.

## Admission Control

Before doing expensive work, decide whether system has capacity.

Could limit:
- concurrent requests;
- jobs per tenant;
- expensive searches;
- upload size.

Rejecting early is often better than accepting work that will time out later.

## Load Shedding

When overloaded, intentionally drop/reject lower-value work.

Examples:
- disable recommendations;
- return cached/stale data;
- reject analytics/debug traffic;
- reject low-priority API calls with 429/503.

Goal:
**preserve core functionality.**

## Graceful Degradation

Examples:

```text
Search suggestions down → search still works
Recommendation service down → show popular items
Like count unavailable → hide count, still show post
```

Not appropriate:
```text
Payment validation down → assume payment succeeded
```

Degradation must respect correctness/security.

## Failure Domains

Avoid correlated failure.

Domains:
- process;
- machine;
- rack/AZ;
- region;
- provider;
- database cluster;
- tenant/resource pool.

Three replicas on one machine do not protect from machine failure.

Two services sharing the same exhausted DB pool are not truly isolated.

## Dependency Isolation

Critical and noncritical dependencies should not have equal power to break requests.

Example checkout:
- payment is critical;
- recommendation is optional;
- analytics is async;
- email is async.

Design path accordingly.

## Health Checks and Overload

A saturated instance may technically be alive but unable to serve traffic.

Readiness should reflect:
- can accept new work?
- critical resources available?

But if every instance marks unhealthy simultaneously due to shared downstream failure, load balancer may remove entire service. Sometimes degraded service is better than no service.

Health checks require careful boundaries.

## Worked Example — Product API + Recommendation Service

Request:
```text
GET /product/123
```

Core data from DB/cache.
Recommendation call optional.

Design:
- recommendation timeout 50ms within request budget;
- no or one bounded retry only if appropriate;
- circuit breaker;
- fallback: omit recommendations;
- separate connection pool/bulkhead;
- monitor recommendation failure and breaker state.

Result:
product page remains available even if recommendation service is slow.

## Small Design Drills

1. Why can a slow dependency be worse than a fast failure?
2. What does jitter solve?
3. Why is retrying a failing service dangerous?
4. What is the difference between circuit breaker and timeout?
5. Give a bulkhead example.
6. What is load shedding?
7. Should checkout fail because analytics is unavailable?

<details>
<summary>Answer key</summary>

1. Slow calls hold resources and create queues/cascading saturation.
2. Prevents synchronized retry waves.
3. Multiplies load during the failure/recovery window.
4. Timeout bounds one call; breaker stops new calls after systemic failure is detected.
5. Separate worker/connection pools for critical vs bulk workload.
6. Intentionally reject/drop lower-priority work to protect core capacity.
7. Usually no; analytics should be async/noncritical.

</details>

## Common Interview Mistakes

- “Retry three times” without timeout/backoff/idempotency.
- Circuit breaker with no timeout.
- Infinite/unbounded queues.
- Health check depends on every optional dependency.
- Fallback that violates correctness.
- Scaling during overload without protecting DB.
- Ignoring retry amplification from clients/gateways.
- No resource isolation between critical/noncritical work.

## Must Remember

- **Slow is a failure mode.**
- **Timeout every remote call.**
- **Retry only transient, safe operations within a deadline.**
- **Use exponential backoff + jitter.**
- **Retries can amplify outages.**
- **Circuit breaker fails fast after systemic dependency failure.**
- **Bulkheads isolate resource pools.**
- **Backpressure prevents unbounded buildup.**
- **Load shedding protects core service.**
- **Graceful degradation must preserve correctness.**

## Interview Revision Summary

For every dependency ask:

```text
Timeout?
Retry?
Idempotent?
Backoff/jitter?
Circuit breaker?
Bulkhead?
Fallback?
What if slow?
What if overloaded?
Queue bounded?
Admission/load shedding?
Failure domain?
```

## Explain Without Notes

A downstream API latency rises from 100ms to 20s while still accepting connections. Explain the cascading failure chain and design defenses.

## Completion Checklist

- [ ] I understand timeouts/deadlines.
- [ ] I design safe retries.
- [ ] I can explain circuit breaker states.
- [ ] I understand bulkheads/backpressure.
- [ ] I can shed/degrade safely.
- [ ] I reason about failure domains.
