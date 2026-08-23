# Lesson 9 — Caching Fundamentals

**Course:** FAANG L4 System Design Interview Course Plan — V2  
**Session:** 9 / 46  
**Recommended time:** 60–90 minutes  
**Phase:** Fundamentals

---

# 1. Lesson Goal

By the end of this lesson, you should understand:

- what caching is;
- why systems use caches;
- where caches can exist;
- cache-aside;
- read-through;
- write-through;
- write-behind;
- TTL;
- eviction policies;
- cache hit rate;
- the relationship between cache and source of truth.

You should also be able to answer four critical questions whenever you introduce a cache:

```text
1. What is the source of truth?
2. What is the cache key?
3. How long can the data be stale?
4. How is the cache invalidated?
```

This lesson focuses on **caching fundamentals**.

Advanced caching problems such as stampedes, hot keys, penetration, and thundering herds are covered in **Lesson 10**.

---

# 2. What Is a Cache?

A cache is a faster storage layer that keeps copies of data that would otherwise be slower or more expensive to retrieve.

Example:

```text
Without cache

Client
  |
  v
Application
  |
  v
Database
```

Every request reaches the database.

With a cache:

```text
Client
  |
  v
Application
  |
  +------> Cache
  |
  +------> Database
```

The application first tries to get the data from the cache.

If the data is already cached:

```text
Application
   |
   v
Cache
   |
   v
Fast response
```

If it is not cached:

```text
Application
   |
   v
Cache miss
   |
   v
Database
   |
   v
Store result in cache
   |
   v
Return response
```

---

# 3. Why Use a Cache?

Caching is usually introduced for one or more of these reasons.

## 3.1 Reduce Latency

Memory access is generally much faster than reading from a remote database or computing a result repeatedly.

Example:

```text
Database query: slower
Cache lookup: faster
```

If a frequently requested product page changes only occasionally, caching its data can improve response latency.

---

## 3.2 Reduce Database Load

Suppose a database receives:

```text
100,000 reads / second
```

If 90% of those requests can be served from cache:

```text
Database receives only about 10,000 reads / second
```

This can significantly reduce pressure on the database.

---

## 3.3 Reduce Expensive Computation

Some values are expensive to calculate.

Examples:

- recommendation results;
- ranked feeds;
- analytics summaries;
- aggregated counters;
- expensive joins.

Instead of recalculating every time:

```text
Compute once
   ↓
Cache result
   ↓
Reuse
```

---

## 3.4 Protect Downstream Services

Caching can reduce the number of calls to:

- databases;
- external APIs;
- internal microservices;
- search clusters;
- object stores.

This helps protect slower or expensive dependencies.

---

# 4. What Should Be Cached?

Caching works best when data is:

- read frequently;
- reused across requests;
- relatively expensive to fetch or compute;
- allowed to be slightly stale;
- small enough to fit in available cache memory.

Examples:

```text
Product details
User profiles
Configuration
Popular posts
Session information
API responses
Computed rankings
```

Caching is less useful when:

- every request is unique;
- data changes continuously;
- stale data is unacceptable;
- the cache hit rate would be very low.

---

# 5. Source of Truth

A cache is usually **not** the source of truth.

Example:

```text
Database = source of truth
Redis = cache
```

If the cache disappears:

```text
Cache lost
   ↓
Application reads database
   ↓
Cache gets rebuilt
```

This distinction is essential.

Whenever you propose a cache in an interview, explicitly state:

> “The database remains the source of truth. The cache stores derived copies for faster reads.”

---

# 6. Cache Hit and Cache Miss

## Cache Hit

The requested value exists in the cache.

```text
Request
   ↓
Cache
   ↓
Value found
```

This is a **cache hit**.

---

## Cache Miss

The requested value does not exist in the cache.

```text
Request
   ↓
Cache
   ↓
Not found
   ↓
Database
```

This is a **cache miss**.

---

# 7. Cache Hit Rate

Cache hit rate measures how often requests are served from the cache.

Formula:

```text
Cache Hit Rate =
Cache Hits / Total Cache Lookups
```

Example:

```text
900 cache hits
100 cache misses

Total lookups = 1000
```

Hit rate:

```text
900 / 1000
= 90%
```

A high hit rate usually means the cache is effectively absorbing read traffic.

But hit rate alone is not enough.

You should also think about:

- latency;
- memory usage;
- stale data;
- database load;
- key distribution.

---

# 8. Where Can a Cache Exist?

Caching can occur at multiple layers.

---

## 8.1 Client-Side Cache

Data is cached on the user's device.

Examples:

- browser cache;
- mobile app cache.

Architecture:

```text
Client Cache
     |
     v
Application Server
```

Advantages:

- extremely fast;
- reduces network traffic.

Disadvantages:

- difficult to invalidate immediately;
- clients may hold stale data.

---

## 8.2 CDN / Edge Cache

Covered more deeply in Lesson 8.

A CDN caches content geographically closer to users.

Useful for:

- images;
- videos;
- CSS;
- JavaScript;
- public static content.

---

## 8.3 Application Local Cache

Each application server keeps data in its own process memory.

Example:

```text
          Load Balancer
            /       \
           v         v
       Server A   Server B
       Local       Local
       Cache       Cache
```

Advantages:

- extremely low latency;
- no network call to cache.

Disadvantages:

- each server has a different cache;
- memory is duplicated;
- invalidation across servers is harder.

---

## 8.4 Distributed Cache

Multiple application servers share a cache.

Example:

```text
         Load Balancer
          /       \
         v         v
    App Server   App Server
         \       /
          \     /
           v   v
      Distributed Cache
             |
             v
          Database
```

Common examples include Redis and Memcached.

Advantages:

- shared cache;
- larger combined capacity;
- consistent cached view across application servers.

Disadvantages:

- requires a network call;
- becomes another distributed dependency;
- needs scaling and failure handling.

---

# 9. Cache-Aside Pattern

Cache-aside is one of the most common caching strategies.

The application explicitly manages the cache.

---

## Read Flow

```text
1. Application receives request.
2. Application checks cache.
3. If found → return cached value.
4. If not found → query database.
5. Store database result in cache.
6. Return result.
```

Diagram:

```text
Application
     |
     v
   Cache
   /   \
 Hit   Miss
 |       |
 v       v
Return  Database
          |
          v
      Store in Cache
          |
          v
        Return
```

---

## Example Pseudocode

```text
function getUser(userId):
    user = cache.get("user:" + userId)

    if user exists:
        return user

    user = database.getUser(userId)

    cache.set("user:" + userId, user, ttl=300)

    return user
```

---

## Advantages

- simple;
- application controls cache behavior;
- only requested data is cached;
- database remains source of truth.

---

## Disadvantages

- first request is slow;
- cache misses hit the database;
- stale data can exist;
- application code becomes responsible for cache management.

---

# 10. Read-Through Cache

With read-through caching, the cache layer is responsible for loading missing data.

Conceptually:

```text
Application
   |
   v
Cache
   |
   +---- Hit → Return
   |
   +---- Miss
           |
           v
        Database
```

The application simply asks the cache.

The cache knows how to fetch data from the underlying storage.

---

## Difference from Cache-Aside

### Cache-Aside

```text
Application handles miss
```

### Read-Through

```text
Cache handles miss
```

The conceptual result is similar, but responsibility differs.

---

# 11. Write-Through Cache

In write-through caching, writes update the cache and underlying storage synchronously.

Example:

```text
Application
   |
   v
Cache
   |
   v
Database
```

A write is considered successful after the underlying storage is updated.

---

## Benefits

- cache remains fresh;
- future reads can hit the cache;
- simpler consistency model than delayed writes.

---

## Costs

- writes are slower;
- every write may update both cache and database;
- unnecessary cache entries may be created for rarely read data.

---

# 12. Write-Behind / Write-Back Cache

In write-behind caching, the application writes to the cache first.

The cache updates the database later.

```text
Application
   |
   v
Cache
   |
   | asynchronous
   v
Database
```

---

## Benefit

Very fast writes.

---

## Risks

If the cache fails before data reaches the database:

```text
Write
  ↓
Cache
  X crash
  ↓
Database never updated
```

Data may be lost.

Therefore write-behind requires careful durability and recovery design.

For many interview systems, this is not the default choice.

---

# 13. Comparing the Main Strategies

| Strategy | Read Miss | Write Behavior | Strength | Main Risk |
|---|---|---|---|---|
| Cache-Aside | Application loads DB | Application usually writes DB and invalidates cache | Simple and common | Stale data |
| Read-Through | Cache loads DB | Depends on implementation | Cleaner application logic | Cache complexity |
| Write-Through | Cache + DB updated together | Synchronous | Fresh cache | Higher write latency |
| Write-Behind | Cache updated first | DB updated asynchronously | Fast writes | Data loss / consistency risk |

For L4 interviews, **cache-aside** is often the most useful default pattern to understand deeply.

---

# 14. Cache Keys

A cache key determines how data is identified.

Example:

```text
user:123
product:500
post:9001
```

Good keys are:

- deterministic;
- easy to construct;
- unique enough;
- stable;
- aligned with access patterns.

---

## Example

Suppose API:

```http
GET /users/123
```

Cache key:

```text
user:123
```

For paginated results:

```text
feed:user:123:cursor:abc
```

But be careful.

Caching many combinations of filters, sorting, and pagination can produce huge numbers of keys.

---

# 15. Time-To-Live (TTL)

TTL defines how long a cached entry stays valid.

Example:

```text
TTL = 5 minutes
```

After five minutes, the entry expires.

---

## Short TTL

Advantages:

- fresher data.

Disadvantages:

- more cache misses;
- more database traffic.

---

## Long TTL

Advantages:

- higher hit rate;
- less database traffic.

Disadvantages:

- stale data lasts longer.

---

## Choosing TTL

TTL should reflect the business requirement.

Examples:

```text
Weather data        → minutes may be acceptable
Product description → minutes/hours may be acceptable
Seat availability   → stale data may be dangerous
User permissions    → stale authorization can be dangerous
```

Do not choose TTL randomly.

Ask:

> How stale is this data allowed to be?

---

# 16. Cache Invalidation

Invalidation means removing or updating stale cached data.

Suppose:

```text
Database:
user:123.name = "Alice"
```

Cache:

```text
user:123.name = "Alice"
```

Then the user changes their name:

```text
Database:
user:123.name = "Alicia"
```

But cache still contains:

```text
Alice
```

The cache is now stale.

---

## Common Invalidation Approaches

### 1. TTL Expiration

Allow the cache to expire naturally.

Simple, but stale data can exist until expiration.

---

### 2. Explicit Invalidation

After database update:

```text
UPDATE database
DELETE cache entry
```

Next read reloads fresh data.

---

### 3. Explicit Cache Update

After database update:

```text
UPDATE database
UPDATE cache
```

This keeps cache fresh but introduces dual-update concerns.

Later lessons will cover distributed workflow failure scenarios in more depth.

---

# 17. Cache Eviction

A cache has limited memory.

When full, it must decide which entries to remove.

This is called eviction.

Common policies include:

---

## LRU — Least Recently Used

Remove entries that have not been accessed recently.

Useful when recently used data is likely to be requested again.

---

## LFU — Least Frequently Used

Remove entries that are used least often.

Useful when some objects are consistently popular.

---

## FIFO — First In, First Out

Remove older inserted entries first.

Simple but does not consider usage patterns.

---

## TTL-Based Expiration

Expired entries become removable.

TTL and eviction policies can work together.

---

# 18. When Caching Can Hurt

Caching adds complexity.

A cache can create:

- stale reads;
- invalidation bugs;
- extra infrastructure;
- memory cost;
- operational complexity;
- failure modes;
- consistency problems.

You should not say:

> “We should cache everything.”

Instead ask:

```text
Is this data frequently reused?
Is retrieval expensive?
Is some staleness acceptable?
Will the cache hit rate be high enough?
```

---

# 19. Example — Product Detail Service

Suppose:

```text
GET /products/123
```

Product data changes infrequently but is read frequently.

Architecture:

```text
Client
  |
  v
Product Service
  |
  v
Cache
  |
  +---- hit → return product
  |
  +---- miss
         |
         v
      Database
         |
         v
      Cache result
```

Possible key:

```text
product:123
```

Possible TTL:

```text
10 minutes
```

When product changes:

```text
1. Update database.
2. Delete product:123 from cache.
3. Next read reloads fresh data.
```

This is a typical cache-aside design.

---

# 20. Example — User Permissions

Suppose:

```text
Can user 123 access account 999?
```

Caching might improve performance.

However, stale permissions can create a security problem.

Imagine access is revoked, but the cached permission remains valid for one hour.

Therefore:

```text
Long TTL
```

may be unacceptable.

Possible choices include:

- very short TTL;
- explicit invalidation;
- not caching the critical authorization decision.

This illustrates an important principle:

> Caching policy is a business correctness decision, not only a performance decision.

---

# 21. Cache Decision Framework

Whenever you add a cache in an interview, answer these questions.

## Question 1 — What are we caching?

Example:

```text
Product object
```

---

## Question 2 — Why cache it?

Example:

```text
Very high read frequency and relatively infrequent updates.
```

---

## Question 3 — What is the source of truth?

Example:

```text
Product database.
```

---

## Question 4 — What is the cache key?

Example:

```text
product:{product_id}
```

---

## Question 5 — What is the TTL?

Example:

```text
5 minutes.
```

---

## Question 6 — How is it invalidated?

Example:

```text
Delete cache entry after successful database update.
```

---

## Question 7 — What happens on a cache miss?

Example:

```text
Read from database and repopulate cache.
```

---

## Question 8 — What happens if cache is unavailable?

Example:

```text
Temporarily fall back to the database, possibly with load protection.
```

Advanced failure behavior will be discussed in Lesson 10 and Lesson 21.

---

# 22. Interview Example

Suppose the interviewer asks:

> How would you reduce database load for frequently accessed user profiles?

A strong answer could be:

```text
User profiles are read much more often than they are updated,
so I would consider a distributed cache in front of the database.

The database remains the source of truth.

I would use a key such as user:{user_id}.

On a read, the service first checks the cache. On a miss,
it reads the database, stores the result with a TTL, and returns it.

When the profile is updated, I would update the database first
and invalidate the corresponding cache key.

The TTL depends on how much staleness the product can tolerate.
```

Notice that this answer explains:

- why;
- source of truth;
- key;
- read behavior;
- update behavior;
- TTL;
- staleness.

That is much stronger than:

> “Use Redis.”

---

# 23. Small Application Drills

## Drill 1 — Product Cache

You have:

```http
GET /products/{product_id}
```

Answer:

```text
What should be cached?

Source of truth:

Cache key:

TTL:

Invalidation strategy:

Behavior on cache miss:
```

---

## Drill 2 — News Article

A news article is read millions of times but may be edited occasionally.

Design a simple caching policy.

```text
Cache location:

Key:

TTL:

Invalidation:

Why caching helps:
```

---

## Drill 3 — Seat Availability

A ticket system shows available seats.

Would you cache seat availability?

Answer:

```text
Yes / No / Partially

Why?

What consistency risk exists?

What TTL would be acceptable?
```

There is no universal answer.

The important part is the reasoning.

---

## Drill 4 — Local vs Distributed Cache

You have 100 stateless application servers.

Compare:

```text
Local cache
```

versus:

```text
Distributed cache
```

Think about:

- latency;
- duplication;
- consistency;
- invalidation;
- network dependency.

---

# 24. Explain the Trade-Offs

For each pair, explain the trade-off aloud.

---

## Short TTL vs Long TTL

```text
Short TTL:
Freshness ↑
Database traffic ↑

Long TTL:
Hit rate ↑
Staleness ↑
```

---

## Local Cache vs Distributed Cache

```text
Local:
Lower latency
Harder consistency across servers

Distributed:
Shared view
Extra network dependency
```

---

## Cache-Aside vs Write-Through

```text
Cache-aside:
Simple and lazy
Potential stale entries / cold misses

Write-through:
Fresher cache
Slower writes and more write work
```

---

# 25. Knowledge Check

Answer without looking back.

### Question 1

What is the main purpose of a cache?

### Question 2

What is the difference between a cache hit and cache miss?

### Question 3

What does cache hit rate measure?

### Question 4

Why should the database usually remain the source of truth?

### Question 5

Describe the cache-aside read flow.

### Question 6

What is the difference between cache-aside and read-through?

### Question 7

What is write-through caching?

### Question 8

What is the main risk of write-behind caching?

### Question 9

What is TTL?

### Question 10

Why is a longer TTL not always better?

### Question 11

What is cache invalidation?

### Question 12

Name two cache eviction policies.

### Question 13

What are the four critical questions you should answer when proposing a cache?

### Question 14

Why might caching authorization data be dangerous?

### Question 15

Why is “Use Redis” an incomplete system design answer?

---

# 26. Suggested Answers

Do not read until you attempt the questions.

---

### Answer 1

To serve frequently reused data faster and reduce load or expensive work on downstream systems.

### Answer 2

A hit means the requested value exists in the cache. A miss means it does not and must be retrieved from another source.

### Answer 3

The proportion of cache lookups successfully served from the cache.

### Answer 4

The cache may expire, evict, lose, or contain stale data. The durable authoritative record should usually live elsewhere.

### Answer 5

Check cache → if hit, return → if miss, read database → store result in cache → return result.

### Answer 6

With cache-aside, the application handles the cache miss and loads the database. With read-through, the cache layer handles loading the missing data.

### Answer 7

Writes update both cache and underlying storage synchronously.

### Answer 8

Data can be lost or become inconsistent if the cache fails before asynchronous persistence finishes.

### Answer 9

Time-to-live: how long a cache entry remains valid before expiration.

### Answer 10

Long TTL improves hit rate but increases how long stale data may be served.

### Answer 11

Removing or updating cached data that is no longer valid.

### Answer 12

Examples: LRU and LFU.

### Answer 13

At minimum:

1. What is the source of truth?
2. What is the cache key?
3. How stale can the data be / what TTL?
4. How is the cache invalidated?

### Answer 14

Revoked or changed permissions may remain cached and incorrectly allow access.

### Answer 15

Because the interviewer needs the reason for the cache, data being cached, source of truth, keys, TTL, invalidation, failure behavior, and trade-offs—not merely a product name.

---

# 27. Spoken Exercise

Without notes, explain:

> “How would I add caching to a read-heavy product service?”

Your explanation should cover:

```text
Why cache
Source of truth
Cache location
Cache key
Cache-aside flow
TTL
Invalidation
Cache miss
Cache failure
Trade-offs
```

Target:

**3–5 minutes**

---

# 28. Mini Design Drill

Scenario:

```text
An application serves public user profiles.

Profiles are read frequently.
Users update their profile a few times per month.
The database is becoming overloaded by profile reads.
```

Design only the caching part.

Your answer should include:

```text
Cache type:
Cache pattern:
Source of truth:
Cache key:
TTL:
Read flow:
Write/update flow:
Invalidation:
Failure behavior:
Main trade-off:
```

Do not design the entire application.

---

# 29. Lesson Completion Checklist

Mark Lesson 9 complete only when you can:

- [ ] Explain what a cache is.
- [ ] Explain why caching can reduce latency.
- [ ] Explain why caching can reduce database load.
- [ ] Distinguish local and distributed caches.
- [ ] Explain cache hit and cache miss.
- [ ] Calculate a basic cache hit rate.
- [ ] Explain cache-aside without notes.
- [ ] Explain read-through.
- [ ] Explain write-through.
- [ ] Explain write-behind and its major risk.
- [ ] Explain TTL and its freshness/load trade-off.
- [ ] Explain cache invalidation.
- [ ] Name at least two eviction policies.
- [ ] Explain why caches are normally not the source of truth.
- [ ] Answer the four critical cache questions.
- [ ] Complete the knowledge check with at least 12/15 correct.
- [ ] Explain a caching strategy aloud in 3–5 minutes.

---

# 30. Session Notes Template

## Concepts I understand well

```text

```

## Concepts still unclear

```text

```

## Three things I want to remember

1.
2.
3.

## Questions to revisit

```text

```

---

# 31. One-Page Recall Sheet

Try to reproduce this from memory before Lesson 10.

```text
CACHING

WHY?
- reduce latency
- reduce DB load
- avoid repeated computation
- protect downstream services

SOURCE OF TRUTH
- usually DB / durable storage
- cache = derived copy

READ
- hit → return
- miss → source of truth → populate cache

PATTERNS
- cache-aside
- read-through
- write-through
- write-behind

KEY
- deterministic
- aligned with access pattern

TTL
- short = fresher, more misses
- long = higher hit rate, more stale data

INVALIDATION
- TTL
- explicit delete
- explicit update

EVICTION
- LRU
- LFU
- FIFO

ALWAYS ASK
1. What is cached?
2. Why?
3. Source of truth?
4. Cache key?
5. TTL?
6. Invalidation?
7. Miss behavior?
8. Failure behavior?
```

---

# Short Revision Note

> **Cache interview rule:** every cache decision should explain **what is cached, why, source of truth, cache key, TTL, invalidation, miss behavior, and failure behavior**.

Cache is a performance optimization, not automatically the authoritative data layer. Freshness requirements determine whether caching is safe and how aggressive the policy can be.

# Topics to Revise

- [ ] cache hit vs miss
- [ ] hit rate
- [ ] local vs distributed cache
- [ ] cache-aside
- [ ] read-through
- [ ] write-through
- [ ] write-behind
- [ ] cache key design
- [ ] TTL
- [ ] invalidation
- [ ] LRU/LFU/FIFO
- [ ] source of truth
- [ ] cache failure fallback

---

# 32. Preview of Lesson 10

Lesson 10 will build on this foundation and cover:

- cache stampede;
- thundering herd;
- hot keys;
- cache penetration;
- negative caching;
- cache warming;
- stale data;
- distributed cache failure;
- strategies for protecting the database when cache behavior becomes pathological.

---

# End of Lesson 9

**Next session:** Lesson 10 — Advanced Caching Problems
