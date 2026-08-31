# Session 9 — Caching Fundamentals

## Outcome

You should be able to decide whether caching is useful, choose an appropriate cache location and strategy, design a cache key and TTL, identify the source of truth, and explain the consistency trade-offs created by caching.

## Why Caching Exists

A cache stores data closer to the consumer or in a faster medium so repeated work can be avoided.

Without cache:

```text
Request → Application → Database
```

With cache:

```text
Request → Application → Cache
                        ├─ hit → return
                        └─ miss → DB → cache → return
```

Caching can reduce:

- latency;
- database load;
- downstream API calls;
- CPU-heavy recomputation;
- network transfer;
- cost.

But caching adds a second copy of data, which creates invalidation and stale-data problems.

## The First Question: What Is the Source of Truth?

A cache should normally not become the authoritative record accidentally.

Example:

```text
PostgreSQL = source of truth
Redis = derived cache
```

If Redis is lost, the system can reconstruct data from PostgreSQL.

This differs from systems where an in-memory store is itself the primary database. Do not call every fast store “just a cache.”

## Cache Locations

### Client cache

Browser/mobile device caches data locally.

Advantages:
- fastest access;
- no server round trip.

Problems:
- hard to invalidate;
- user may be offline;
- sensitive data must be handled carefully.

### CDN / edge cache

Best for geographically distributed delivery of cacheable HTTP/media content.

### Application-local cache

Each app instance holds an in-process cache.

Advantages:
- extremely fast;
- no network call.

Problems:
- each replica has its own copy;
- low aggregate hit rate after scaling;
- invalidation across nodes is difficult;
- memory disappears when process restarts.

### Distributed cache

Shared cache such as Redis/Valkey/Memcached-style service.

Advantages:
- shared among app instances;
- central TTL/invalidation;
- larger aggregate capacity.

Costs:
- network hop;
- another dependency;
- clustering/failure considerations.

## Cache-Aside

Most common interview pattern.

### Read

```text
1. Read cache.
2. Hit → return.
3. Miss → read database.
4. Put result in cache.
5. Return.
```

### Write

Typically:
1. update source of truth;
2. invalidate/update cache.

The application controls caching explicitly.

### Advantages

- simple;
- cache only requested data;
- cache failure can often degrade to DB.

### Problems

- miss penalty;
- stale data;
- race conditions during invalidation;
- cache stampede.

## Read-Through

Application asks cache. Cache itself loads missing data from backing store through an integrated loader.

Application sees one abstraction.

Useful but more infrastructure-specific.

## Write-Through

Write goes through cache and synchronously updates backing store.

Advantages:
- cache and store can remain aligned;
- reads often hit fresh cache.

Cost:
- write latency includes backing store;
- caching layer becomes part of write path.

## Write-Behind / Write-Back

Write updates cache first; backing store is updated asynchronously.

Advantages:
- fast writes;
- can batch writes.

Risks:
- data loss if cache fails before flush;
- ordering/consistency complexity;
- cache becomes durability-sensitive.

Use only when workload tolerates the semantics.

## TTL

TTL defines how long a cached entry remains valid.

Choose TTL from acceptable staleness, not habit.

Examples:

```text
Static country list → hours/days
Product details → minutes
Inventory count → very short or avoid cache for authoritative decision
User profile → perhaps minutes + invalidation
```

Long TTL:
- higher hit rate;
- lower backend load;
- more stale risk.

Short TTL:
- fresher;
- more misses/backend traffic.

## Eviction

When cache capacity is full, entries must be removed.

Common conceptual policies:
- LRU-like;
- LFU-like;
- TTL expiration;
- random/implementation-specific variants.

For interview purposes, focus on the consequence:
**cache memory is finite**, so hot/valuable data should remain more often than cold data.

## Cache Hit Rate

```text
hit rate = cache hits / total cache lookups
```

If 90% of 10k RPS hits cache, DB sees roughly 1k RPS from this path, ignoring writes/other queries.

If hit rate suddenly drops to 50%, DB load becomes ~5k RPS.

This is why cache hit rate is an operational metric.

## Cache Keys

A cache key must uniquely represent the data variant.

Examples:

```text
product:{productId}
profile:{userId}
feed:{userId}:{cursor/version}
```

Bad key:
```text
product
```
when many products exist.

Also include dimensions that affect output:
- locale;
- authorization/tenant;
- version;
- query/filter where needed.

But adding irrelevant dimensions lowers hit rate.

## Cache Invalidation

“There are only two hard things...” jokes aside, invalidation is genuinely difficult because multiple copies exist.

Strategies:

### TTL-only
Allow data to age out.

Good when bounded staleness is acceptable.

### Invalidate on write
Update DB, then delete cache entry.

Next read repopulates.

### Update cache on write
Update both DB and cache.

Can reduce misses but creates dual-write/race concerns.

### Versioned keys
Change key/version when data changes.

Useful for immutable assets or versioned datasets.

## Race Example

Two readers and one writer:

```text
Reader A misses cache
Reader A reads old DB value
Writer updates DB + invalidates cache
Reader A writes old value into cache
```

Now stale data has been resurrected.

Possible mitigations:
- short TTL;
- version numbers;
- write invalidation ordering;
- delayed second invalidation in some architectures;
- avoid caching correctness-critical rapidly changing data.

The best strategy depends on required consistency.

## What Should Not Be Cached?

Avoid or carefully design caching for:
- highly volatile correctness-critical state;
- one-time secrets;
- authorization results without proper keying/invalidation;
- data with almost no reuse;
- huge objects with poor cost/benefit.

Example:
A seat reservation decision should check authoritative inventory/lock state, not trust a 30-second cached seat count.

## Worked Example — Product Details

Requirements:
- product metadata changes a few times/day;
- 20k read RPS;
- 200 write RPS;
- users tolerate ~30 seconds stale description;
- price should update quickly.

Design:

```text
GET product
  ↓
Distributed cache key product:{id}
  ├─ hit
  └─ miss → Product DB → cache TTL 60s
```

On write:
- transactionally update DB;
- invalidate `product:{id}`.

For price, perhaps use a shorter TTL or explicit invalidation event.

Source of truth remains DB.

## Comparison Table

| Strategy | Read miss | Write behavior | Main trade-off |
|---|---|---|---|
| Cache-aside | App loads | DB + invalidate/update | simple, miss/staleness races |
| Read-through | Cache loads | separate | clean abstraction, cache dependency |
| Write-through | Usually warm | sync cache→store | higher write latency |
| Write-behind | warm | async persist | fast writes, durability complexity |

## Small Design Drills

1. Why can an application-local cache have lower hit rate after scaling from 2 to 20 instances?
2. A cache hit ratio drops from 95% to 50%. Why might the DB fail even though request traffic did not change?
3. Which is the source of truth in a normal cache-aside design?
4. Why is TTL not a correctness guarantee?
5. Should a payment balance be cached for a final debit decision?
6. What dimensions might belong in a cache key for a localized product page?

<details>
<summary>Answer key</summary>

1. Requests for the same key are spread across many independent caches; each must warm separately.
2. Miss traffic to DB increases dramatically.
3. The backing durable data store.
4. Data can be stale during the TTL; races may also write stale values.
5. Usually authoritative balance/transaction state should be checked in the source of truth for correctness.
6. Product ID plus locale/currency/tenant/version if those alter the representation.

</details>

## Common Interview Mistakes

- Saying “add Redis” before identifying access patterns.
- Forgetting source of truth.
- Giving every key the same arbitrary TTL.
- Ignoring cache failure.
- Caching correctness-critical state casually.
- Updating DB and cache without discussing dual-write races.
- Assuming 100% hit rate.
- Forgetting key dimensions and privacy/tenant separation.

## Must Remember

- **A cache is normally a derived copy; identify the source of truth.**
- **Cache-aside is the common default mental model.**
- **TTL trades freshness for hit rate/backend load.**
- **Cache keys must represent all meaningful response variants.**
- **Cache invalidation creates race conditions.**
- **Distributed cache is another network dependency.**
- **Hit-rate drop can suddenly overload the database.**
- **Do not cache correctness-critical rapidly changing data without a clear consistency model.**

## Interview Revision Summary

Before adding cache, answer:

```text
What is expensive?
What is reused?
Source of truth?
Cache location?
Key?
TTL?
Read strategy?
Write/invalidation strategy?
Acceptable staleness?
What if cache fails?
What if hit rate drops?
```

## Explain Without Notes

Design caching for a product-catalog endpoint with 95% reads, frequent price changes, multiple locales, and a relational source of truth.

## Completion Checklist

- [ ] I understand local/distributed/CDN caches.
- [ ] I can explain cache-aside/read-through/write-through/write-behind.
- [ ] I can choose a TTL based on staleness.
- [ ] I understand invalidation races.
- [ ] I monitor hit rate and fallback capacity.
