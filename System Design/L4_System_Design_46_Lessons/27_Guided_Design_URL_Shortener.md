# Session 27 — Guided Design — URL Shortener

## Interview Prompt

> Design a URL-shortening service similar to Bitly. Users submit a long URL and receive a short URL. Visiting the short URL should redirect quickly to the original URL.

### Rules

Before reading the reference reasoning, spend **35–45 minutes** designing it yourself.

Your design should cover:
- requirements;
- useful estimates;
- APIs;
- data model;
- alias generation;
- redirect path;
- caching;
- expiration;
- availability;
- analytics separation.

### Requirement Change

Later, support:
1. custom aliases;
2. alias expiration;
3. prevention of unsafe alias reuse after expiration.

---

# STOP — Attempt the Design First

Do not continue until you have drawn a complete first attempt.

---

# Reference Reasoning

## 1. Clarify Requirements

Core functional requirements:

- create short URL from long URL;
- redirect short alias to long URL;
- optionally expire aliases;
- custom alias support;
- basic deletion/disable may be optional.

Non-goals:
- full analytics dashboard initially;
- crawling destination content;
- link preview generation.

Non-functional priorities:

- redirect should be low latency;
- redirect path highly available;
- mapping must be durable;
- reads dominate writes;
- alias uniqueness is correctness-critical;
- analytics may be eventually consistent.

## 2. Useful Scale Assumptions

Assume:

- 100M new URLs/month;
- 100:1 redirect-to-create ratio;
- peak redirect traffic 50k RPS;
- average stored URL + metadata ~1 KB;
- aliases retained for years unless expired.

Storage per month:

```text
100M × 1 KB ≈ 100 GB/month
```

Redirect traffic is far more important than create traffic, so caching and simple point lookup matter.

Do not overfit exact numbers.

## 3. APIs

Create:

```http
POST /links
{
  "url": "https://example.com/very/long/path",
  "customAlias": "optional",
  "expiresAt": "optional"
}
```

Response:

```json
{
  "alias": "aZ91xK",
  "shortUrl": "...",
  "expiresAt": "..."
}
```

Redirect:

```http
GET /{alias}
```

Response:
- HTTP redirect such as 301/302 depending on mutability/caching semantics.

If destinations may change, permanent browser caching with 301 can be undesirable; 302/307-style semantics may preserve control. State the trade-off.

## 4. Data Model

Authoritative mapping:

```text
Link
----
alias          PK/UNIQUE
destination
created_at
expires_at
owner_id       optional
status
generation_id / tombstone metadata if reuse policy requires it
```

Primary read:

```text
alias → destination
```

This naturally fits:
- relational DB with unique alias index;
- distributed key-value store at very large scale.

The access pattern is simple enough that either can be defended.

## 5. Alias Generation Options

### Option A — Database sequence + Base62

Generate unique integer:
```text
12500123
```

Encode:
```text
Base62 → "Q7az"
```

Advantages:
- collision-free if sequence is unique;
- compact;
- simple.

Problems:
- central sequence coordination;
- aliases predictable.

Mitigate predictability with permutation/obfuscation if required.

### Option B — Random Base62

Generate fixed-length random alias.

Advantages:
- decentralized;
- less predictable.

Problem:
- collision possible.

Use:
```text
generate → conditional insert UNIQUE(alias)
collision → retry
```

At low occupancy, collision retry is rare with sufficient namespace.

### Option C — Snowflake-style ID + encoding

Distributed roughly ordered unique IDs, encoded shorter.

Adds worker/clock complexity; useful if create throughput/regions justify it.

### Recommendation

At L4, choose the simplest option matching requirements.

For moderate single-region create traffic:
- DB sequence + encoding is perfectly defensible.

For globally distributed create:
- random or distributed ID generation may be preferable.

## 6. High-Level Architecture

```text
                 ┌──────────────┐
Create requests →│ API Service  │→ Mapping DB
                 └──────────────┘

Redirect requests
       ↓
 CDN/Edge optional
       ↓
 Load Balancer
       ↓
 Redirect Service
       ↓
 Distributed Cache
       ├─ hit → redirect
       └─ miss → Mapping DB → cache
```

Analytics:

```text
Redirect Service → async event/stream → Analytics pipeline
```

Do not update an analytics row synchronously on every redirect if that harms the critical path.

## 7. Redirect Flow

```text
GET /abc123
  ↓
check cache
  ↓ hit
verify not expired/disabled
  ↓
return redirect
```

Miss:

```text
cache miss
  ↓
DB lookup by alias
  ↓
not found/expired → 404/410
  ↓
cache mapping / negative result briefly
  ↓
redirect
```

Use negative caching carefully to protect repeated invalid aliases.

## 8. Cache Design

Key:
```text
link:{alias}
```

Value:
```text
destination + expiry/status/version
```

TTL:
- bounded by link expiration;
- popular immutable mappings can cache long;
- custom destination edits require invalidation/versioning.

Hot alias:
- viral URL can become a hot cache key.
- CDN/edge redirect caching may help if semantics allow.
- local cache for extremely hot immutable mappings can reduce one distributed-cache-node pressure.

## 9. Expiration

At redirect:
- check `expires_at`.

Cleanup can be lazy:
- expired mapping returns not found/expired;
- background job deletes old data later.

Do not require real-time deletion at expiration if logical check suffices.

## 10. Custom Alias

User asks for:
```text
/my-company
```

Need atomic uniqueness:

```sql
INSERT alias
UNIQUE(alias)
```

Do not:
```text
check if free
then insert
```
without DB uniqueness because concurrent creators race.

Reserved aliases:
```text
admin
api
login
robots.txt
...
```
should be blocked.

## 11. Unsafe Alias Reuse

Suppose:
1. `abc` points to bank.com;
2. shared everywhere;
3. link expires;
4. new user gets `abc` pointing to malicious.example.

Old bookmarks now redirect to unrelated content.

This is unsafe alias reuse.

Options:

### Never reuse aliases
Simplest and safest.

Keep tombstone:
```text
alias → retired forever
```

### Long quarantine
Reuse only after very long period, still risky.

### Namespace by owner/version
Changes URL semantics.

For public short links, **never reuse retired aliases** is a strong default.

Tombstone can store minimal metadata rather than full destination forever.

## 12. Analytics

Redirect service emits:

```text
LinkVisited(alias, timestamp, coarse metadata)
```

Stream/queue consumers aggregate:
- clicks;
- country/device;
- time window.

Analytics can lag and should not block redirect.

Sampling may be acceptable at extreme scale if requirements permit.

## 13. Failure Scenarios

### Cache down
Fallback to DB, but DB may not handle full redirect traffic.
- load shed abusive invalid aliases;
- local emergency cache;
- restore gradually;
- ensure DB/read replicas have headroom.

### DB leader down
Redirect may still serve cached mappings temporarily.
Create operations may fail/degrade until failover.

### Analytics down
Redirect continues; events buffered/dropped according to analytics durability requirement.

### Viral link
Hot key + bandwidth/load.
Use edge caching/local replicas and protect DB.

## 14. Multi-Region

If global redirect latency matters:
- replicate mapping data to regional read stores;
- CDN/edge cache;
- create writes have home/primary region;
- new-link propagation may lag slightly unless stronger requirement.

Custom alias uniqueness across regions becomes coordination problem. Options:
- route alias creation to one authority/home;
- globally consistent key store;
- partition alias namespace.

## 15. Trade-Offs

| Choice | Benefit | Cost |
|---|---|---|
| Sequence ID | no collisions, compact | central/predictable |
| Random ID | decentralized/unpredictable | collision retry |
| Long cache TTL | low DB load | stale edit risk |
| Edge caching | very low redirect latency | invalidation/control |
| Never reuse alias | safety | namespace consumed forever |
| Async analytics | fast redirect | delayed metrics |

## Interviewer Follow-Up Questions

Answer aloud:

1. Why not generate a random 6-character string and assume no collision?
2. What happens if the cache cluster dies?
3. How would you support destination editing?
4. How do you prevent malicious users from reserving system aliases?
5. How do you make alias creation globally unique?
6. What breaks at 10× redirect traffic?
7. Why separate analytics?

## Common Mistakes

- No unique constraint on alias.
- Analytics write in redirect transaction.
- No alias reuse policy.
- Saying “Redis makes it fast” without cache failure.
- Random alias without collision handling.
- URL stored in CDN as source of truth.
- Ignoring expiration.
- Sharding before workload justifies it.
- 301 used without considering destination mutability.

## Repair Drills

If weak on IDs:
- compare sequence, random Base62, Snowflake.

If weak on caching:
- inject cache failure and viral alias.

If weak on correctness:
- simulate concurrent custom-alias creation.

## Must Remember

- **Primary access pattern is alias → destination.**
- **Alias uniqueness is authoritative correctness, ideally enforced atomically.**
- **Redirect path should stay tiny and highly cacheable.**
- **Analytics belongs off the critical path.**
- **Random aliases need collision detection.**
- **Expired public aliases should usually not be reused.**
- **Cache failure must not instantly destroy the database.**

## Self-Score

Score yourself 0–4 on:
- Requirements
- Estimation
- APIs/Data
- HLD/Flows
- Scalability
- Correctness
- Reliability
- Security/Abuse/Cost
- Trade-offs
- Communication

Record top two weaknesses before moving on.
