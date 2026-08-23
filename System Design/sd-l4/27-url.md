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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 28 — Design a Distributed Rate Limiter
