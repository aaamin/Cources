# Session 10 — Advanced Caching Problems

## 1. Must Learn

### Staleness & invalidation
- **Understand:** Understand stale data after source updates and why invalidation is hard.
- **Decision/trade-off:** Freshness vs hit rate/complexity.

### Stampede / thundering herd
- **Understand:** Know many concurrent misses/expirations can hammer the backend.
- **Decision/trade-off:** Simple independent misses vs request coalescing/locking/stale serving.

### Hot keys
- **Understand:** Understand one highly popular key can overload a cache node or backend.
- **Decision/trade-off:** Partitioned distribution vs replication/local caching/special handling.

### Cache penetration
- **Understand:** Know repeated requests for nonexistent/uncacheable keys can bypass caching.
- **Decision/trade-off:** Correct negative results vs stale absence and memory use.

### Negative caching
- **Understand:** Cache not-found/empty results briefly to protect backends.
- **Decision/trade-off:** Backend protection vs delayed visibility of newly created data.

### TTL randomization & warming
- **Understand:** Know synchronized expirations are dangerous and selected hot data may be prewarmed.
- **Decision/trade-off:** Operational work/memory vs smoother load.

### Cache failure & eviction pressure
- **Understand:** Understand sudden hit-rate collapse when cache fails or evicts aggressively.
- **Decision/trade-off:** Graceful degradation vs backend overload.

## 2. Should Know

- Request coalescing/single-flight conceptually.
- Stale-while-revalidate conceptually for data allowed to be slightly stale.
- Differentiate hot key from generally high traffic.

## 3. Recognition Only

- Bloom filter as one penetration-defense option
- Multi-tier cache strategies

## 4. Important Comparisons

- Stampede vs penetration vs hot key.
- Negative caching vs not caching misses.
- Fixed TTL vs randomized TTL.
- Fresh fetch vs stale-while-revalidate.
- Failing cache vs degraded cache hit rate.

## 5. Important Interview Questions

1. What happens when thousands of entries expire together?
2. How do I protect the database from a cache stampede?
3. What if one cache key receives extreme traffic?
4. What if clients repeatedly request nonexistent keys?
5. What happens when the cache cluster becomes unavailable?
6. When is stale-while-revalidate acceptable?

## 6. Common Interview Mistakes

- **Treating every cache problem as TTL** → Identify the actual failure mode first.
- **Same TTL on huge key populations** → Add jitter/randomization when synchronized expiry could spike load.
- **No backend protection on cache outage** → Use admission control, stale data, rate limits, or other degradation as appropriate.
- **Ignoring nonexistent-key traffic** → Use validation/negative caching or similar protection.
- **Assuming sharding solves hot keys** → A single hot key can still concentrate load.

## 7. Communication

### Important Vocabulary

cache invalidation, stale data, stampede, thundering herd, cache penetration, hot key, negative caching, cache warming, randomized TTL, request coalescing, stale-while-revalidate

### Useful Interview Phrases

- “The danger is synchronized misses amplifying load onto the database.”
- “I’d add TTL jitter so many keys do not expire at the same instant.”
- “For this hot key, I need to reduce concentration rather than merely add more shards.”

### Important Questions to Ask the Interviewer

- **Question:** “How stale may responses be during overload?”  
  **Why it matters:** Determines stale-serving options.
- **Question:** “Is popularity highly skewed?”  
  **Why it matters:** Determines hot-key handling.
- **Question:** “Can nonexistent-key traffic be abusive?”  
  **Why it matters:** Determines penetration defenses.

## 8. ⭐ Must Remember

1. Stampede = many requests miss together.
2. Penetration = requests repeatedly bypass cache for missing/uncacheable data.
3. Hot key = concentrated traffic on one key.
4. Randomized TTL reduces synchronized expiry.
5. Negative caching can protect missing-key lookups.
6. Cache failure must not instantly overwhelm the database.

## 9. Study Priority

1. Study first: stampede, hot keys, penetration.
2. Study next: invalidation/staleness, negative caching, TTL jitter.
3. Finish with: cache failure, request coalescing, stale-while-revalidate.

## 10. Revision Checklist

- [ ] Differentiate major cache failure modes.
- [ ] Mitigate synchronized misses.
- [ ] Handle a hot key.
- [ ] Protect missing-key traffic.
- [ ] Explain cache-outage degradation.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
