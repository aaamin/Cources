# Session 28 — Design a Distributed Rate Limiter

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice atomic shared state, algorithm choice, sharding, global versus local enforcement, and failure policy.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Algorithms: fixed window, sliding-window counter/log concept, token bucket, leaky bucket.
- Dimensions: per-user, per-IP, per-tenant, per-endpoint, global.
- State: counters/tokens, TTL, atomic updates, clock/time-window issues.
- Placement: gateway/service/client-side local limiter combinations.
- Distribution: partition keys, local caches, centralized/shared store, regional limits.
- Failure policy: fail-open versus fail-closed depending endpoint risk.
- Response: 429, retry-after, remaining quota if exposed.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Do a timed design using token bucket or sliding-window counter and justify the choice.
- [ ] Change request: enforce per-user + tenant + global quotas across regions.
- [ ] Explicitly choose fail-open or fail-closed for a login endpoint and a public read API.

## **Must Remember for the Interview**

- **The algorithm must match the product semantics; 'Redis counter' alone is not a rate-limiter design.**
- **Atomic state updates matter under concurrency.**
- **Global limits across regions trade accuracy, latency, and availability.**
- **Fail-open/fail-closed is a product/security decision.**
- **Shard by the rate-limit key and plan for hot/global keys.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Token bucket permits bursts up to bucket capacity while enforcing average rate.**
- **Sliding windows are more accurate but can cost more state/compute.**
- **Use atomic increment/token update + TTL/time semantics.**
- **Regional approximation may be acceptable; strict global quotas require coordination.**
- **Always state failure policy.**

## Self-Test Before Marking This Session Complete

- [ ] Did I choose and justify an algorithm?
- [ ] Did I define key granularity?
- [ ] Did I discuss atomicity and TTL/time?
- [ ] Did I discuss multi-region accuracy?
- [ ] Did I explicitly state fail-open/fail-closed?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support per-user, per-tenant, and global quotas across regions.

**Score using the 40-point rubric.**


---

**Progress:** Session 28/46  
**Next:** Session 29
