# Lesson 10 — Advanced Caching Problems

**Phase:** Fundamentals  
**Session:** 10/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn the failure patterns that appear when caches become popular: stampedes, hot keys, cache penetration, stale data, cold starts, and cache outages.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Cache stampede / thundering herd

A stampede happens when many requests miss the same key at once and all query the backing store. A classic trigger is simultaneous expiration of a very popular key. Mitigations include single-flight/request coalescing, locking refresh, stale-while-revalidate, proactive refresh, and randomized TTL so hot keys do not all expire together.

## Hot keys

A hot key receives disproportionate traffic. Total cache capacity can look healthy while one node melts because all requests for one key land there. Mitigations include per-process local caching, replication of very hot values, key splitting for aggregations, or redesigning the access pattern.

## Cache penetration

Repeated queries for values that do not exist always miss and can hit the database every time. Negative caching stores “not found” for a short TTL. Bloom filters can reject impossible keys before DB access when the domain supports it.

## Stale data and invalidation races

Database and cache updates are separate operations. If a process crashes between them, cache state may disagree with the source of truth. A common write path is `update DB → invalidate cache`; even this can race under concurrency, so freshness guarantees must be explicit.

## Cache warming

After restart, failover, or deploying a new cluster, the cache is cold. If full traffic falls through immediately, the database can collapse. Preload hot data, ramp traffic gradually, or rate-limit fallback while the cache rebuilds.

## Cache outage

A cache is a dependency that can fail. If normal hit rate is 95%, losing it may increase database reads by 20×. Protect the DB with load shedding, request limits, degraded responses, and enough fallback capacity.

## Worked example — popular configuration expires

A configuration object is read on every request and expires every 60 seconds. Thousands of instances miss together. Improve it with staggered TTL, local + shared caching, a single refresher, and stale-while-revalidate. The goal is to turn synchronized misses into controlled refresh.

## Interview lens

Advanced cache questions test whether you understand that caches can concentrate load as well as reduce it. Always discuss protection of the source of truth.

## What to remember

Remember the recurring failure modes: stampede, hot key, penetration, stale state, cold start, and fallback overload.

## Review after reading

1. What causes a stampede?
2. Why can one hot key overload one cache node?
3. What is negative caching?
4. Why can cache loss take down the DB?
5. How does TTL jitter help?

## Deeper study notes

### Stampede controls trade freshness for protection

Serving slightly stale data for a short period can be safer than allowing thousands of requests to overload the database. `stale-while-revalidate` is a deliberate availability/performance choice: one request refreshes while others keep receiving the old value. Whether that is acceptable depends on business semantics.

### Hot-key mitigation is workload-specific

For immutable/config data, replicate the value into each process. For counters, split writes across subkeys and aggregate. For a celebrity profile, cache at several layers. For a lock-like key, replication does not help because updates require coordination. Do not propose one universal “hot key solution.”

### Negative caching needs bounded TTL

Caching “not found” protects against repeated invalid IDs, but a long TTL can hide a newly created resource. Choose a shorter TTL and invalidate when creation is known. Security-sensitive endpoints may also need request validation/rate limits because negative caching alone does not stop random-key attacks.

### Cache consistency is usually intentionally weaker

If an operation absolutely cannot return stale state—seat ownership, permissions in a sensitive workflow—the simplest solution may be to bypass cache for the authoritative decision. Caching everything is not an objective.

### Common mistakes

- Setting the same TTL on millions of keys loaded together.
- Falling back unlimited traffic to DB after cache failure.
- Treating cache as source of truth without durability design.
- Using a global lock around every cache miss, turning cache refresh into a bottleneck.


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

**Next:** Lesson 11 — SQL & Relational Databases
