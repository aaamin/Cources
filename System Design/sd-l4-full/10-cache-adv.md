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


## Important interview ideas

> **Important:** A cache is not only a performance accelerator; it can become a traffic amplifier when it fails. Always design the fallback path, not just the hit path.

### Stampede control patterns

Suppose a popular key expires. There are several strategies:

1. **Single-flight/request coalescing** — one request loads the key, others wait.
2. **Lock around refresh** — one owner refreshes; lock must have timeout/failure handling.
3. **Stale-while-revalidate** — serve slightly stale data while one background refresh runs.
4. **Probabilistic/early refresh** — refresh before expiry based on remaining TTL.
5. **TTL jitter** — randomize expiration so many keys do not expire simultaneously.

The correct choice depends on how stale the data may be and whether waiting is acceptable.

### Hot key mitigation

A distributed cache normally maps one key to one partition. For an extremely hot read-only value, you can replicate it into local process caches or several derived keys. For counters, split writes across multiple subkeys and aggregate later if approximate/async results are acceptable.

Do not blindly “add cache nodes.” More nodes do not automatically split one hot key.

### Negative caching and abuse

If an attacker requests random nonexistent IDs, every request can miss and hit the database. Cache “not found” briefly:

```text
missing:user:xyz → NOT_FOUND, TTL 30s
```

The TTL should be short enough that a newly created object becomes visible soon. Bloom filters can reject impossible keys at large scale, but negative caching is simpler for many systems.

### Cache consistency races

Consider cache-aside update:

```text
writer updates DB
writer deletes cache
```

A concurrent reader can race: it may read old DB data before the write commits and populate stale cache after invalidation. TTL eventually repairs it, but strict freshness may require versioning, write-through, serialization, or event-driven invalidation.

At L4, recognizing the race and matching mitigation to required consistency is enough.

## Failure scenario — complete cache loss

Imagine the cache serves 95% of 100k read QPS. Database normally receives 5k QPS. Cache cluster disappears.

Naive fallback sends 100k QPS to a database sized for perhaps 10k. The database fails, turning a cache outage into a total outage.

Protection can include:

- rate limiting/load shedding;
- stale local cache;
- request coalescing;
- gradual cache rewarming;
- database headroom;
- degraded responses for non-critical data.

## Interview questions and model answers

### Q1. “What is a cache stampede?”

Many requests simultaneously miss the same key(s) and all query the source of truth. It is especially common after expiration or cold restart. I would prevent synchronized refresh with single-flight, stale-while-revalidate, TTL jitter, or proactive warming.

### Q2. “How do you handle a celebrity hot key?”

For read-heavy immutable/slow-changing data, put a small local cache on app instances or replicate the value across cache keys/nodes. If it is a write-heavy counter, use sharded counters and aggregate, depending on freshness needs.

### Q3. “Can we just fall back to DB if Redis dies?”

Functionally yes, but capacity may make that unsafe. I would quantify the normal cache hit rate and ensure fallback is bounded through rate limits, graceful degradation, or enough database headroom.

### Q4. “How do you keep cache and DB consistent?”

First define allowed staleness. Commonly the DB is source of truth; writes update DB then invalidate cache. TTL is a safety net. Stronger freshness may require versioned values, synchronized write-through, or invalidation events. There is no universal perfect invalidation strategy.

## Common mistakes to avoid

- No cache-outage story.
- Same TTL on millions of keys.
- Assuming more cache nodes solve a hot key.
- Caching nonexistent values forever.
- Ignoring write/read races.
- Using a cache as authoritative state accidentally.
- Rewarming at full traffic with no source protection.

## Short revision note

Advanced cache failures: **stampede, hot key, penetration, stale race, cold start, total outage**. The source of truth must survive or be protected when cache efficiency collapses.

## Topics to revise

- [ ] stampede / thundering herd
- [ ] single-flight
- [ ] stale-while-revalidate
- [ ] TTL jitter
- [ ] hot keys
- [ ] negative caching
- [ ] invalidation races
- [ ] cache outage fallback

## Interview-ready synthesis

### A strong 60–90 second explanation

I treat the cache as a dependency that can fail catastrophically. I design stampede protection, hot-key handling, negative caching, safe invalidation, warmup, and bounded fallback so loss of cache does not overload the source of truth. The amount of staleness the product tolerates determines which technique is safe.

### How this topic connects to the wider system

- Performance: cache misses can synchronize into a source-of-truth spike.
- Reliability: fallback must be capacity-limited rather than unlimited DB traffic.
- Correctness: invalidation races can serve stale values even after a write.
- Scalability: a single hot key can overload one partition regardless of cluster size.

### Revision flashcards with answers

**Stampede?**  
Many concurrent misses for the same/related keys all hit the source.

**TTL jitter?**  
Randomize expirations so many keys do not expire at once.

**Negative cache?**  
Cache a short-lived NOT_FOUND result to prevent repeated misses for nonexistent keys.

**Hot key mitigation?**  
Local cache/replication/key-splitting depending on read vs write pattern.

**Why not unlimited DB fallback?**  
The database may be sized assuming high cache hit rate and collapse under full traffic.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Product catalog: hot product TTL expiry can stampede the DB; stale-while-revalidate keeps page available.
- Celebrity profile: one hot key may need local per-instance cache because adding cluster nodes does not split one key.
- Invalid IDs attack: negative caching prevents repeated nonexistent lookups from reaching the database.

### When not to overuse this idea

Do not solve every freshness problem with a very long TTL. If stale data can violate authorization or inventory correctness, invalidate or avoid caching that decision.

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

**Next:** Lesson 11 — SQL & Relational Databases
