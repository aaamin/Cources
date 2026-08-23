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


## Detailed reference design

### Clarify enforcement scope

Ask whether limits are:

- per IP;
- per authenticated user;
- per API key/tenant;
- per endpoint;
- global across regions;
- burstable or strict.

Also ask the failure policy. A login brute-force limiter may prefer **fail closed**. A low-risk read endpoint may **fail open** if limiter storage is unavailable.

### Algorithms

#### Fixed window

```text
key: user123:2026-08-23T18:35-minute
count++
```

Simple, but a client can send `N` requests at the end of one window and `N` immediately at start of next.

#### Sliding log/window

Track request timestamps or buckets over the last duration. Fairer but more memory/CPU.

#### Token bucket

State:

```text
tokens
last_refill_time
```

Tokens refill at rate R up to capacity B. A request consumes cost C. This naturally supports short bursts while enforcing long-term average.

For interviews, token bucket is a strong default when burst tolerance matters.

### Atomic decision

Two gateways must not both read `tokens=1` and both allow a request. The counter/token update must be atomic at the owner:

```text
read/refill/check/decrement
```

performed as one transaction/script/conditional operation.

### Architecture

```text
Client
  ↓
API Gateway
  ↓
Rate-limit check
  ↓
Fast distributed state store
  ↓ allowed
Backend Service
```

For lower latency and higher availability, add local limiters:

```text
local bucket for burst
    +
central/global quota for coarse correctness
```

This creates bounded approximation but removes one remote call from every request.

### Key design

Example:

```text
rl:{tenant}:{endpoint}
```

If one tenant dominates traffic, its counter becomes a hot key. Strict centralized counters inherently serialize updates for that key. Mitigations include local allocation of quota chunks, hierarchical limits, or accepting approximate distributed counters.

### Multi-region problem

A strict global limit of 1000 req/s across 10 regions requires coordination. Synchronously coordinating every request adds cross-region latency.

Alternatives:

1. allocate each region a share of quota;
2. periodically rebalance unused quota;
3. use local burst limits + global slower enforcement;
4. choose a home region for strict account-level limits.

This is a classic **accuracy vs latency/availability** trade-off.

### Response contract

A rejected request should return enough information for a good client:

```text
HTTP 429
Retry-After
remaining quota / reset time (optional)
```

Avoid revealing sensitive anti-abuse internals if security requirements say otherwise.

## Failure walkthrough

### State store unavailable

Decide by endpoint:

- payment/abuse endpoint → fail closed or conservative local limit;
- ordinary content read → fail open with emergency local cap.

### Clock skew

If clients provide timestamps, attackers can manipulate them. Use server-side time. Multiple server clocks can differ; token algorithms should tolerate small skew or use datastore/server time.

### Redis/state latency rises

Rate limiter becomes system bottleneck. Apply local caching/allocation, timeout, circuit breaker, and conservative fallback.

### Hot tenant

One key saturates one partition. Hierarchical/local quotas reduce central atomic update rate.

## Interviewer follow-ups

### “How would you limit 100 requests/min?”

I would first ask burst semantics. For strict simple policy, fixed/sliding window may be enough. If bursts are allowed while average is controlled, token bucket is better.

### “Can we use a local in-memory counter?”

For one server, yes. Across many servers it only limits each instance independently unless traffic is sticky. It can be part of a hierarchical design but not a strict shared quota by itself.

### “What about different request costs?”

Use weighted tokens: cheap GET costs 1, expensive export costs 100. The policy service maps operation to cost.

### “How do you change policies?”

Store policy/config separately and cache it at gateways. Counter state remains in the fast data plane. Version policy so all gateways converge safely.

## Common interview mistakes

- Algorithm named with no burst semantics.
- Non-atomic get/check/set.
- No fail-open/fail-closed choice.
- “Global rate limit” with no cross-region coordination trade-off.
- Client clock trusted.
- One hot key ignored.
- Rate limiter state treated as durable business data unnecessarily.

## Short revision note

**Rate limiter pattern:** policy key + atomic counter/token state + low-latency enforcement + explicit failure policy + hierarchy for global scale.

## Topics to revise

- [ ] fixed/sliding/token bucket
- [ ] atomic updates
- [ ] rate-limit keys
- [ ] Retry-After / 429
- [ ] hot keys
- [ ] local vs central limiter
- [ ] multi-region quota
- [ ] fail-open vs fail-closed

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll first clarify the quota key, window/burst semantics, and fail-open/fail-closed policy. Then I’ll choose an algorithm such as token bucket, ensure the state update is atomic, and discuss how global limits trade accuracy for cross-region latency.

## How the design evolves at 10×

At 10×, the remote atomic state store may become the bottleneck. Introduce hierarchical/local buckets, quota leasing, and policy caching. For strict global limits, isolate only the small set that truly needs cross-region coordination.

## Quick revision flashcards

**Token bucket state?**  
tokens plus last refill time, updated atomically.

**Why local limiter?**  
Removes remote check from every request but makes global quota approximate unless coordinated.

**Fail closed where?**  
Security/abuse-sensitive endpoint where allowing excess is worse than temporary rejection.

**Hot key?**  
One tenant/global key may serialize; use quota allocation/hierarchy.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 29 — Design Pastebin
