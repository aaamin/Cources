# Session 9 — Caching Fundamentals

## 1. Must Learn

### Cache role & source of truth
- **Understand:** Understand cache as a derived fast copy and identify authoritative storage.
- **Decision/trade-off:** Latency/load reduction vs staleness and extra complexity.

### Local vs distributed cache
- **Understand:** Know process-local cache is fastest but inconsistent per instance; distributed cache is shared but adds network dependency.
- **Decision/trade-off:** Latency/simplicity vs shared consistency/capacity.

### Cache patterns
- **Understand:** Understand cache-aside, read-through, write-through, and write-behind conceptually.
- **Decision/trade-off:** Application control/simplicity vs consistency, write latency, and failure complexity.

### TTL & invalidation
- **Understand:** Know how entries expire or are actively removed when data changes.
- **Decision/trade-off:** Freshness vs hit rate/complexity.

### Cache key design
- **Understand:** Keys must encode the identity/variant/tenant dimensions required for correctness.
- **Decision/trade-off:** Reuse/hit rate vs collision/leakage/fragmentation.

### Eviction & hit rate
- **Understand:** Understand bounded cache capacity, eviction, and why hit rate affects backend load.
- **Decision/trade-off:** Memory cost vs backend protection.

### Failure/miss behavior
- **Understand:** Know what happens on miss and when cache is unavailable.
- **Decision/trade-off:** Graceful fallback vs database overload.

## 2. Should Know

- Serialization/storage overhead and object-size effects.
- Cache warming only when cold-start cost is significant.
- Write-through/write-behind should be justified; cache-aside is a common interview baseline.

## 3. Recognition Only

- LRU/LFU policy names
- Near-cache patterns

## 4. Important Comparisons

- Local vs distributed cache.
- Cache-aside vs read-through.
- Write-through vs write-behind.
- Short TTL vs long TTL.
- Cache hit vs miss path.

## 5. Important Interview Questions

1. What is the source of truth?
2. What exactly is the cache key?
3. How stale may the value be?
4. How is cache invalidated after writes?
5. What happens on cache miss?
6. What happens if the cache cluster is unavailable?

## 6. Common Interview Mistakes

- **“Add Redis” with no policy** → Define key, source of truth, TTL, invalidation, and miss behavior.
- **Treating cache as authoritative accidentally** → Keep source-of-truth ownership clear.
- **No cache failure path** → Protect the backend from sudden miss amplification.
- **Wrong key scope** → Include tenant/user/variant dimensions needed for correctness.
- **Caching low-reuse data** → Cache only when reuse/latency/load benefit justifies complexity.

## 7. Communication

### Important Vocabulary

cache, source of truth, cache-aside, read-through, write-through, write-behind, TTL, eviction, cache hit rate, cache key, local cache, distributed cache

### Useful Interview Phrases

- “The database remains the source of truth; the cache is derived state.”
- “I’d use cache-aside here because it keeps the write path simple.”
- “The acceptable staleness determines my TTL and invalidation strategy.”

### Important Questions to Ask the Interviewer

- **Question:** “How stale may this data be?”  
  **Why it matters:** Determines TTL/consistency policy.
- **Question:** “Is this data reused across many requests?”  
  **Why it matters:** Determines whether caching is worthwhile.
- **Question:** “Can the source-of-truth store handle a cache outage?”  
  **Why it matters:** Determines protection/fallback design.

## 8. ⭐ Must Remember

1. Always name the source of truth.
2. A cache needs a key, freshness policy, miss path, and failure path.
3. Cache-aside is a strong simple baseline.
4. TTL trades freshness for hit rate.
5. High hit rate reduces backend load.
6. A cache outage can become a database outage if fallback is uncontrolled.

## 9. Study Priority

1. Study first: source of truth, cache-aside, local vs distributed.
2. Study next: keys, TTL, invalidation.
3. Finish with: eviction, hit rate, miss/failure behavior.

## 10. Revision Checklist

- [ ] Choose where caching belongs.
- [ ] Define source of truth and cache key.
- [ ] Choose a cache pattern.
- [ ] Explain TTL/invalidation.
- [ ] Handle miss and cache outage safely.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
