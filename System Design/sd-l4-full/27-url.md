# Lesson 27 — Design a URL Shortener

**Phase:** Guided Design  
**Session:** 27/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Create a short URL for a long URL.
- Redirect a short code to the original URL.
- Optional expiration/custom aliases.
- Low redirect latency and high availability.

## 2. Scale and workload shape

Assume a read-heavy service. Example: 10M new URLs/day and 100M redirects/day. Redirect QPS is about 10× create QPS, so the lookup path is the main optimization target. Storage is mostly small key→URL mappings; analytics can be much larger but should not slow redirects.

## 3. API / contract surface

```http
POST /v1/urls
GET /{code}
```

`POST` accepts the long URL plus optional custom alias/expiry and returns the chosen code. `GET /{code}` performs the redirect. Custom aliases require atomic uniqueness, not a separate “check then insert” race.

## 4. Data model

```text
ShortURL(
  code PRIMARY KEY,
  long_url,
  owner_id?,
  created_at,
  expires_at?
)
```

The main access pattern is direct lookup by `code`. The mapping store is the source of truth. Click analytics should be emitted asynchronously rather than written on the redirect critical path.

## 5. High-level architecture

```text
Client
  ↓
Load Balancer / Edge
  ↓
URL Service
  ├─ Cache
  └─ Mapping Store
        ↓
   async click event
```

Create path: validate → generate/reserve code → persist mapping → return. Redirect path: cache lookup → store on miss → return 301/302.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Short-code generation

Random base62 strings reduce coordination but need collision handling. A sequence or distributed ID generator guarantees uniqueness but adds coordination and may reveal volume. The code-space size should keep collision probability low.

### Redirect caching

Cache by short code. Popular links may be very hot; local caches can absorb extreme repeated reads. The DB remains authoritative.

### Custom aliases

Use a unique constraint/conditional write. Reuse after deletion needs an explicit policy because old copies of the URL may exist indefinitely.

## 7. Failure modes and recovery

- Cache outage: fall back to the store but rate-limit/fail safely so DB does not receive a 20× spike.
- Store unavailable: cached redirects can still work; misses fail.
- Create request retried: an idempotency key prevents duplicate ownership/billing effects.
- Expired mapping: treat as not found even if cleanup is delayed.
- Analytics pipeline down: redirects continue; buffer/drop analytics according to requirements.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Start with one relational/key-value store plus cache. Partition only when mapping volume/write load requires it. Multi-region active-active is unnecessary unless global write/availability requirements justify the complexity.

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

### Capacity sketch

If there are 100M redirects/day, average redirect rate is only around 1.2k/s, but a popular link or product launch can concentrate far more. The important shape is **read-heavy + highly skewed by key**. The mapping dataset itself is small compared with media systems: even billions of URL records can be manageable with compact storage, while redirect latency and availability dominate.

### Create flow

```text
1. Authenticate/validate request if accounts are in scope.
2. Normalize/validate the long URL according to product policy.
3. If custom alias: atomically reserve it.
4. Else generate candidate code; retry collision if random.
5. Persist mapping durably.
6. Optionally warm cache.
7. Return short URL.
```

Do not publish a short link before its mapping is durable, or users can receive a URL that immediately fails.

### Redirect flow

```text
GET /abc123
→ edge/service
→ cache lookup
→ mapping store on miss
→ check expiry
→ populate cache
→ return redirect response
→ emit click event asynchronously
```

The click event belongs after the redirect decision because analytics loss is usually less harmful than redirect latency/failure.

### 301 vs 302 reasoning

Permanent redirects can be cached aggressively by browsers/proxies, reducing your traffic but also reducing future control and analytics visibility. Temporary redirects keep more requests flowing through your service. The correct choice depends on product requirements; explaining the trade-off matters more than memorizing one status.

### Common interview mistakes

- Designing analytics synchronously in the redirect path.
- Using a central counter for every ID at global scale without discussing coordination.
- Saying “random string means no collision” instead of handling collision probability.
- Reusing expired/deleted aliases without considering old bookmarks/caches.
- Adding sharding before one store or managed KV system actually needs it.

### Reusable patterns learned

This system teaches: read-heavy key lookup, cache-aside, unique-ID generation, hot-key handling, asynchronous analytics, and separating correctness-critical state from derived telemetry.


## Detailed reference design

### Requirements clarification

A clean interview scope is:

**Functional**
- create a short URL;
- redirect short code to original URL;
- optional custom alias and expiration.

**Non-functional**
- redirect latency should be very low;
- mappings should be durable;
- service should be highly available;
- custom alias creation must preserve uniqueness.

**Non-goals for first version**
- full analytics dashboard;
- malware scanning pipeline;
- ad platform;
- custom domains.

This scope keeps the core problem centered on **ID generation + high-scale key lookup**.

### Back-of-the-envelope reasoning

Assume 10M new links/day and 100M redirects/day.

```text
writes ≈ 10M / 1e5 sec ≈ 100/s average
reads  ≈ 100M / 1e5 sec ≈ 1,000/s average
```

A 10× peak is still moderate for a distributed service. Storage matters more over years: if each mapping averages ~1 KB including metadata/index overhead, 10M/day is ~10 GB raw/day or several TB/year before replication.

The architectural conclusion is not “we need 100 shards immediately.” It is:

- workload is read-heavy;
- cache gives strong ROI;
- key lookup is simple;
- storage grows continually;
- code uniqueness must be correct.

### Create flow

```text
Client
  ↓ POST /v1/urls
API/LB
  ↓
URL Service
  ├─ validate URL / policy
  ├─ allocate or generate code
  └─ conditional insert into mapping store
          ↓
      success → return short URL
```

For custom aliases, use a unique constraint/conditional insert. Do not do:

```text
SELECT alias
if missing → INSERT
```

because two writers can race.

### Redirect flow

```text
GET /abc123
   ↓
Edge/LB
   ↓
URL Service
   ↓
Cache: code → destination
  ├─ hit → redirect immediately
  └─ miss → mapping DB → cache → redirect
```

Popular codes may be cached at the application/edge layer. The mapping DB remains source of truth.

### 301 vs 302 interview note

A permanent redirect (301-style semantics) can be cached aggressively by browsers/CDNs, reducing service traffic but making destination updates harder to reflect. A temporary redirect (302-style) keeps more control/analytics at the service. You do not need to debate exact HTTP behavior deeply; state the product trade-off.

### Code generation options

**Random base62**
- decentralized generation;
- large key space;
- collisions handled by retry/conditional insert;
- harder to enumerate sequentially.

**Sequence → base62**
- guaranteed uniqueness if allocator is correct;
- compact codes;
- central/partitioned ID allocation needed;
- exposes approximate creation volume/order.

**Distributed ID generator**
- high throughput;
- more moving pieces;
- often unnecessary at small scale.

At L4, compare two choices and choose one based on requirements.

### Partitioning strategy

If the mapping store eventually needs sharding, hash by short code. Redirect requests already contain the code, so routing is naturally single-shard. Range partitioning by creation time would create write hotspots and does not help lookup.

### Expiration

Store `expires_at`. Reads should reject an expired mapping even if cleanup is delayed. A background sweeper can delete old rows/cache entries asynchronously.

This separation is useful:

> **expiration correctness happens on read; cleanup is an optimization.**

### Analytics separation

Do not synchronously write analytics to the critical redirect database.

```text
redirect success
   ↓
async event
   ↓
analytics stream/warehouse
```

If analytics is unavailable, redirects should still work.

## Failure and overload walkthrough

### Cache fails

Redirect traffic falls to the DB. If cache normally absorbs 95% of traffic, the DB may overload. Use load shedding, local cache, gradual rebuild, and sufficient headroom.

### Mapping DB leader fails

Cached mappings still redirect. Cache misses/creates may fail briefly until database failover. This is a good example of graceful partial availability.

### Code collision

Conditional insert fails; generate another code and retry. Collision handling belongs in the write path, not after returning success.

### Hot viral link

One code can become a hot cache key. Local in-process caching or CDN redirect caching may spread reads instead of sending every lookup to one distributed-cache partition.

## Interviewer follow-ups

### “How would you support link editing?”

Keep code stable and update destination in source of truth. Invalidate cache. If browsers/CDNs cache permanent redirects, updates propagate more slowly—one reason to choose redirect semantics carefully.

### “How do you prevent enumeration?”

Random non-sequential codes make enumeration harder, but this is not authorization. Private links need access control or unguessable tokens with sufficient entropy and possibly expiration.

### “What breaks at 100×?”

First inspect cache QPS/hot keys, mapping-store size/partition capacity, and edge bandwidth. The stateless URL service is easy to scale horizontally. If DB size/write throughput crosses one-node capacity, hash-shard by code.

## Common interview mistakes

- Overengineering with Kafka/Cassandra/multi-region before basic flow.
- No uniqueness story for aliases.
- Cache as source of truth.
- No hot-link handling.
- Synchronous analytics on redirect path.
- ID strategy stated with no collision/coordination discussion.
- Sharding by timestamp despite lookup by code.

## Short revision note

**URL shortener pattern:** simple key lookup, read-heavy cache, correct unique code creation, hash partitioning if needed, async analytics, explicit redirect/cache semantics.

## Topics to revise

- [ ] functional/non-functional scope
- [ ] QPS/storage estimate
- [ ] base62/random IDs
- [ ] uniqueness/conditional insert
- [ ] cache-aside redirect
- [ ] 301 vs 302 trade-off
- [ ] hash sharding by code
- [ ] expiration/cleanup
- [ ] viral hot key
- [ ] async analytics

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll focus on creating a short mapping and low-latency redirect. I’ll treat redirect as read-heavy, keep the mapping store authoritative, and separate analytics from the critical path. The two areas I expect to deep-dive are ID uniqueness and cache/hot-key behavior.

## How the design evolves at 10×

At 10× redirects, stateless redirect servers scale easily; inspect cache throughput and viral hot keys first. At 10× stored mappings/writes, hash-partition by code and distribute ID generation. Add multi-region reads only if global latency/availability requires it.

## Quick revision flashcards

**Source of truth?**  
Durable code→URL mapping store.

**Best shard key?**  
Short code, because redirect already supplies it.

**Why async analytics?**  
Click analytics must not increase redirect latency/availability coupling.

**Hot link?**  
Local/edge cache or replicated hot value; more cache nodes alone may not split one key.

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

**Next:** Lesson 28 — Design a Distributed Rate Limiter
