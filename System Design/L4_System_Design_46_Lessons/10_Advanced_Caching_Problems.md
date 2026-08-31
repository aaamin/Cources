# Session 10 — Advanced Caching Problems

## Outcome

You should be able to recognize and mitigate cache stampedes, penetration, hot keys, synchronized expiry, cold-cache failures, negative caching issues, cache unavailability, and stale-data trade-offs.

## Why This Matters

A cache makes the happy path faster, but it can also concentrate risk. A database sized for 5% miss traffic may collapse when the cache fails or thousands of requests recompute the same missing key.

Think of cache resilience as:

```text
Normal hits
  ↓
What happens when hit rate changes?
  ↓
Can origin survive?
```

## Cache Stampede / Thundering Herd

Suppose a popular key receives 20k RPS and its TTL expires.

```text
Key expires
   ↓
20k requests miss
   ↓
20k DB queries/recomputations
```

Even though only one fresh value is needed.

### Mitigations

#### Request coalescing / single flight
One request performs the expensive refresh; others wait or use stale value.

```text
Many misses
  ↓
One loader
  ↓
Cache updated
  ↓
Waiting requests use result
```

#### Lock around rebuild
Distributed lock/lease can limit refreshers, but lock failure and stale holders need care.

#### Stale-while-revalidate
Serve recently stale data while one process refreshes.

Excellent when bounded staleness is acceptable.

#### Early refresh
Refresh popular keys before hard expiry.

#### Randomized TTL / jitter
Avoid many keys expiring simultaneously.

## Synchronized Expiry

If 1M keys are populated during deployment with exactly a 1-hour TTL:

```text
At +60 min → huge simultaneous expiry
```

Add TTL jitter:

```text
base 60 min ± random interval
```

This spreads misses over time.

## Cache Penetration

Requests repeatedly target keys that do not exist.

If “not found” is not cached:

```text
attacker/random IDs
     ↓
cache miss every time
     ↓
database
```

### Negative caching

Cache “not found” for a short TTL.

Benefits:
- protects DB from repeated misses.

Risks:
- newly created data may temporarily appear absent;
- attacker can still generate unbounded unique keys.

Other controls:
- input validation;
- rate limiting;
- Bloom filter recognition in some systems, but not required for L4;
- authorization before expensive lookup.

## Hot Keys

One key receives extreme traffic.

Examples:
- celebrity profile;
- viral post;
- World Cup live score;
- global configuration key.

Problems:
- one cache node/shard may saturate;
- network bandwidth to that node;
- lock contention;
- eviction of other data.

### Mitigations

- application-local replicas of ultra-hot immutable data;
- CDN/edge cache;
- replicate hot item;
- split/duplicate key with read fan-out strategy where safe;
- longer TTL;
- pre-warm;
- protect origin.

Partitioning more cache nodes does not automatically solve one hot key because the key still maps to one partition unless replicated.

## Cache Warming

Populate expected hot data before traffic arrives.

Useful:
- deploy/migration;
- known event start;
- popular catalog;
- region launch.

Risks:
- wasted work on unused items;
- backend load during warming;
- stale warm data.

## Cold Cache

After cache restart/region failover, hit rate can be near zero.

This creates a “cache recovery storm.”

Mitigations:
- gradual traffic shift;
- pre-warm;
- request coalescing;
- fallback rate limiting;
- origin capacity headroom;
- persistent cache where appropriate;
- serve stale replicas if safe.

## Cache Failure

If distributed cache is unavailable:

### Fail through to DB
Simple, but DB may be unable to handle full traffic.

### Fail request
Protects DB but reduces availability.

### Local fallback
Use short-lived in-process cache for selected data.

### Shed non-critical traffic
Protect core operations.

The correct behavior depends on the endpoint.

Example:
- product descriptions may degrade to stale/local data;
- bank transfer should not use stale balance as truth.

## Eviction Storm

A cache under memory pressure evicts hot entries, producing more DB misses, which may increase latency and cause more in-flight requests.

Monitor:
- memory;
- evictions;
- hit rate;
- latency;
- DB fallback QPS.

## Stale Data

Not all stale data is equally harmful.

Acceptable:
- like count 10 seconds behind;
- product recommendation;
- feed ranking.

Potentially dangerous:
- seat availability decision;
- account balance used for debit;
- permission revocation.

Define staleness per field/operation.

## Cache Invalidation Races

Example:

```text
T1 Reader: cache miss
T2 Reader: DB returns V1
T3 Writer: DB writes V2
T4 Writer: invalidates cache
T5 Reader: writes V1 to cache
```

Possible techniques:
- versioned values;
- shorter TTL;
- compare versions before cache set;
- invalidate after transaction;
- event-driven invalidation;
- avoid caching highly contentious state.

There is no universal perfect invalidation recipe.

## Distributed Cache Partition Failure

A sharded cache may lose one node:
- only a subset of keys miss;
- remapping can move many keys;
- failover replica may be used.

Consistent hashing can reduce movement, but hot key and consistency issues remain.

## Worked Example — Viral Post

Normal:
- post page gets 100 RPS;
- cache TTL 5 min.

Viral:
- 100k RPS.

If one cache key maps to one node, that node may become bottleneck.

Possible design:
- CDN cache for public post content;
- local cache on app nodes;
- separate dynamic counters;
- stale-while-revalidate;
- request coalescing;
- longer TTL for immutable post body;
- database protected from direct stampede.

Separate data by freshness:
- post body: cache long;
- like count: eventually consistent / short TTL;
- viewer-specific permission: do not accidentally share-cache.

## Small Design Drills

1. Why does adding more cache shards not necessarily fix a hot key?
2. Why use random TTL jitter?
3. What is negative caching?
4. Cache is down and DB can handle only 20% of normal traffic. What options exist?
5. Why can a cache restart cause an outage even when the cache itself recovers quickly?
6. Which is safer to serve stale: weather forecast or account authorization?

<details>
<summary>Answer key</summary>

1. One key still usually maps to one shard, so its traffic remains concentrated.
2. To prevent mass simultaneous expiration/reload.
3. Briefly caching absence/not-found to prevent repeated misses.
4. Shed/degrade noncritical traffic, local/stale fallback, rate limit, gradual recovery, prioritize core paths.
5. Cold cache sends the full miss load to origin.
6. Weather is generally safer; authorization can be security-critical.

</details>

## Common Interview Mistakes

- Naming “cache stampede” without explaining mechanism.
- Saying “use distributed lock” without stale lock/failure considerations.
- Assuming DB can absorb full cache miss traffic.
- Forgetting cold-start behavior.
- Caching all fields with one freshness policy.
- Treating hot key as only a sharding problem.
- Negative-caching nonexistent data for too long.
- Ignoring cache eviction metrics.

## Must Remember

- **Stampede = many requests rebuild the same missing/expired data.**
- **Coalesce refreshes or serve stale where safe.**
- **Jitter TTLs to avoid synchronized expiry.**
- **Negative caching protects repeated nonexistent-key lookups.**
- **A single hot key can overload one cache node despite many shards.**
- **Cold cache can overload origin.**
- **Design cache failure behavior before production.**
- **Staleness tolerance is operation-specific.**

## Interview Revision Summary

Failure checklist:

```text
Popular key expires?
Many keys expire together?
Random nonexistent keys?
One key becomes viral?
Cache restarts?
Cache node fails?
Hit rate drops?
Evictions rise?
Can origin survive?
Can stale data be served?
```

## Explain Without Notes

Your Redis cluster normally has a 98% hit rate. It restarts during a traffic peak. Explain the failure chain and at least five mitigations.

## Completion Checklist

- [ ] I can explain stampede and coalescing.
- [ ] I understand negative caching.
- [ ] I can reason about hot keys.
- [ ] I use TTL jitter.
- [ ] I design cold-cache/cache-down behavior.
- [ ] I separate freshness requirements by operation.
