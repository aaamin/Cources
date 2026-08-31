# Session 21 — Reliability, Overload & Failure Isolation

## 1. Must Learn

### Timeouts
- **Understand:** Bound how long callers wait for dependencies.
- **Decision/trade-off:** Giving slow dependencies time vs tying up resources and propagating latency.

### Retries with backoff & jitter
- **Understand:** Retry transient failures cautiously; exponential backoff and jitter avoid synchronized pressure.
- **Decision/trade-off:** Recovery probability vs extra load/duplicate work.

### Circuit breakers
- **Understand:** Stop repeatedly calling a failing/slow dependency temporarily.
- **Decision/trade-off:** Fast failure/protection vs rejecting requests that might have succeeded.

### Bulkheads & failure domains
- **Understand:** Isolate resource pools/components so one failure does not consume everything.
- **Decision/trade-off:** Efficiency/shared capacity vs stronger isolation.

### Backpressure & admission control
- **Understand:** Slow/reject incoming work when downstream capacity is exhausted.
- **Decision/trade-off:** Controlled degradation vs unbounded backlog/collapse.

### Load shedding & graceful degradation
- **Understand:** Drop lower-priority work or reduce features to preserve core service.
- **Decision/trade-off:** Feature completeness vs system survival.

### Cascading failure & capacity protection
- **Understand:** Understand slow dependencies, retry storms, queue growth, and resource exhaustion can propagate.
- **Decision/trade-off:** Availability under stress vs aggressive retry/overcommit.

## 2. Should Know

- Health checks and fail-fast behavior.
- Retries should have budgets/limits and usually apply only to retry-safe operations.
- Capacity protection must consider the bottleneck/downstream, not only the caller.

## 3. Recognition Only

- Hedged requests
- Adaptive concurrency control

## 4. Important Comparisons

- Timeout vs retry.
- Retry immediately vs backoff+jitter.
- Circuit breaker vs retry.
- Backpressure vs load shedding.
- Shared resource pool vs bulkhead isolation.
- Graceful degradation vs total failure.

## 5. Important Interview Questions

1. What happens when a dependency is slow, not down?
2. When does retry improve reliability and when does it worsen overload?
3. What should be shed first under overload?
4. How can one dependency failure be prevented from consuming all resources?
5. Which health signal should remove an instance from traffic?
6. What happens when queues keep growing?

## 6. Common Interview Mistakes

- **Retrying immediately and indefinitely** → Use bounded retries with backoff/jitter and retry safety.
- **No timeout** → Unbounded waits consume resources.
- **Retrying non-idempotent operations blindly** → Address duplicate effects.
- **Circuit breaker as magic** → Define fallback/degraded behavior.
- **Scaling callers into a saturated dependency** → Protect the bottleneck with backpressure/admission.

## 7. Communication

### Important Vocabulary

timeout, retry, exponential backoff, jitter, circuit breaker, bulkhead, backpressure, admission control, load shedding, graceful degradation, retry storm, cascading failure, failure domain

### Useful Interview Phrases

- “A slow dependency is dangerous because it consumes resources while requests accumulate.”
- “Retries need a budget and backoff; otherwise they amplify overload.”
- “I’d shed lower-priority work to preserve the core path.”

### Important Questions to Ask the Interviewer

- **Question:** “Which operations are safe to retry?”  
  **Why it matters:** Determines retry/idempotency policy.
- **Question:** “What functionality can degrade first?”  
  **Why it matters:** Defines overload survival strategy.
- **Question:** “Which dependency is the tightest capacity bottleneck?”  
  **Why it matters:** Determines where backpressure belongs.

## 8. ⭐ Must Remember

1. Timeout every remote dependency appropriately.
2. Retries can amplify failures.
3. Use backoff + jitter + bounded retry.
4. Circuit breakers protect against repeated failing calls.
5. Isolate failures with bulkheads/failure domains.
6. Backpressure/load shedding are survival mechanisms.
7. Design for slow dependencies, not only hard failures.

## 9. Study Priority

1. Study first: timeouts, retries/backoff/jitter.
2. Study next: circuit breakers, bulkheads, cascading failures.
3. Finish with: backpressure, admission, load shedding, graceful degradation.

## 10. Revision Checklist

- [ ] Handle a slow dependency.
- [ ] Design bounded safe retries.
- [ ] Explain circuit breaker and bulkhead roles.
- [ ] Protect downstream capacity.
- [ ] Describe graceful degradation under overload.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
