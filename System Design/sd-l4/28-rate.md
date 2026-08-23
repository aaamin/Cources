# Lesson 28 — Design a Distributed Rate Limiter

**Phase:** Guided Design  
**Session:** 28/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Limit by user, IP, API, or tenant.
- Support burst-friendly policies.
- Check on the request path with very low latency.
- Define behavior if limiter state is unavailable.

## 2. Scale and workload shape

The limiter must handle at least the peak request rate of the protected APIs. State per key is tiny, but access is extremely frequent and can be skewed by abusive clients. The design should not turn rate limiting into the new bottleneck.

## 3. API / contract surface

Conceptual internal contract:

```text
allow(key, policy, cost) → allowed, remaining, retry_after
```

The gateway/service passes a stable key such as `tenant:123:/search`. The result can map to HTTP 429 with a retry hint.

## 4. Data model

For token bucket:

```text
key → current_tokens, last_refill_time
```

For window counters:

```text
(key, window_start) → count
```

Updates must be atomic. Expiration removes old window state.

## 5. High-level architecture

```text
Client
  ↓
API Gateway
  ↓
Rate Limiter
  ↓
Atomic Counter / Fast State Store
  ↓ if allowed
Backend Service
```

At very high scale, local limiters can enforce burst protection while a shared/global layer controls coarser quotas.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Algorithms

Fixed window is simple but permits boundary bursts. Sliding-window log is accurate but expensive. Sliding-window counters approximate with less state. Token bucket allows controlled bursts while enforcing a long-term rate.

### Atomic state

Two app instances checking the same user must not both read the same old count and over-allow. Use atomic increment/compare/script operations.

### Regional/global limits

Strict global quota requires cross-region coordination and adds latency. A practical alternative divides a global budget into regional budgets with occasional reconciliation.

## 7. Failure modes and recovery

- State store slow: limiter can cascade latency into every API call; use short timeouts.
- Hot key: one abusive/global key overloads one shard; split hierarchical policies or cache/localize checks.
- Clock skew: time-window calculations disagree; prefer server/store time or algorithms tolerant of skew.
- Store unavailable: choose fail-open for low-risk reads or fail-closed for sensitive auth/payment endpoints.
- Retry storms: rate limiting should cooperate with client backoff.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Centralized counters maximize accuracy but create latency/dependency cost. Local counters maximize availability but are approximate. Hybrid designs are common: local fast-path + periodic/global enforcement.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Fixed window example

For 100 requests/minute, store `count(user, minute_bucket)`. It is cheap, but a client can send 100 requests at 12:00:59 and another 100 at 12:01:00. The two fixed windows allow 200 requests in two seconds. This may be acceptable for coarse quotas but not for strict smoothing.

### Token bucket mechanics

A bucket has capacity `B` and refill rate `r`. Each request consumes tokens. If enough tokens exist, allow and subtract; otherwise reject. A full bucket permits a burst of up to `B`, while refill enforces the sustained rate. This is why token bucket is common for APIs that want burst tolerance.

### Placement

Rate limiting can happen at CDN/edge, API gateway, individual service, or several layers. Edge limits cheaply block abusive traffic early. Service-level limits can use business identity and endpoint-specific semantics. A layered design may apply IP protection at edge and tenant quota at service.

### Multi-region accuracy

If every request synchronously updates one global counter, latency and availability suffer. Alternatives:

- allocate each region part of the global quota;
- allow small overshoot and reconcile;
- use local burst buckets plus slower global accounting;
- route a tenant consistently to a home region.

The tighter the global guarantee, the more coordination you pay for.

### Common interview mistakes

- Choosing an algorithm without explaining burst behavior.
- Performing read-then-write instead of atomic counter mutation.
- Ignoring limiter-store failure and fail-open/fail-closed policy.
- Treating IP address as reliable user identity for every use case.
- Forgetting that rate-limit state itself can be a hot-key workload.

### Reusable patterns learned

Atomic counters, approximate vs strict global state, fail-open/fail-closed, hierarchical quotas, hot-key handling, and edge admission control.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 29 — Design Pastebin
