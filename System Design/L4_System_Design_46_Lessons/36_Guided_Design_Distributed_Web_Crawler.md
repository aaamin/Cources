# Session 36 — Guided Design — Distributed Web Crawler

## Interview Prompt

> Design a distributed web crawler that discovers URLs, fetches pages, deduplicates them, stores content/metadata, respects robots/politeness, and periodically recrawls.

Change request:
> Prioritize news freshness without overloading small websites.

Attempt **40–50 minutes** first.

---

# STOP — Attempt First

Cover:
- URL frontier;
- canonicalization;
- dedup;
- per-host politeness;
- robots;
- fetch workers;
- retry;
- storage;
- recrawl;
- priority/freshness;
- distributed scheduling.

---

# Reference Reasoning

## 1. Requirements

Core:
- seed URLs;
- discover links;
- fetch pages;
- respect robots policy;
- deduplicate URLs/content;
- per-host rate limits;
- retry transient errors;
- store content/metadata;
- recrawl changed pages.

Non-functional:
- huge scale;
- do not overload external sites;
- durable frontier;
- fair host scheduling;
- crawler failures/restarts safe.

Change:
- news sites get higher freshness, but small sites remain protected.

## 2. Pipeline

```text
Seeds
  ↓
URL Frontier / Scheduler
  ↓
Fetch Queue
  ↓
Fetch Workers
  ↓
Parse/Extract
 ├→ Content Store
 ├→ Metadata/Index
 └→ Discovered URLs → Normalize/Dedup → Frontier
```

Critical subsystem is **frontier scheduling**, not HTTP fetch alone.

## 3. URL Canonicalization

Same logical page can appear as:
```text
HTTP vs HTTPS
trailing slash
fragments
tracking query params
hostname case
```

Canonicalization rules:
- normalize host;
- remove fragment;
- normalize default ports;
- carefully handle query parameters;
- follow canonical tags/redirects as signals.

Do not over-normalize and merge distinct pages.

## 4. URL Dedup

Before enqueueing:
```text
Have we seen this canonical URL?
```

At scale:
- distributed key-value set;
- hash/fingerprint;
- Bloom filter as optional recognition to reduce storage lookups, with false positives caveat.

Exact visited state may be retained with timestamps for recrawl.

## 5. Content Dedup

Different URLs can host same/similar content.

Hash page body:
```text
content_hash
```

Exact duplicate:
- store one blob/reference;
- avoid duplicate indexing.

Near-duplicate detection is advanced and optional.

## 6. Robots / Politeness

Crawler should respect site policies.

Need per-host/domain state:
```text
robots rules
crawl delay / next_allowed_time
active requests
```

Do not let 1000 workers simultaneously hit one small site.

## 7. Host-Based Scheduling

Naive global queue:
```text
example.com/a
example.com/b
example.com/c
...
```

Many workers can hammer same host.

Better frontier:
- partition URLs by host;
- maintain per-host queue;
- scheduler releases host when allowed.

Concept:

```text
HostQueue A ─┐
HostQueue B ─┼→ Polite Scheduler → Fetch workers
HostQueue C ─┘
```

Per-host concurrency/rate control is essential.

## 8. DNS / Connections

Repeated DNS/connect cost can be optimized:
- DNS cache;
- connection reuse;
- host-local fetching.

But respect DNS TTL/change; do not cache forever.

## 9. Retry

Transient:
- timeout;
- 5xx;
- connection reset.

Use:
- bounded backoff;
- respect Retry-After;
- reduce host rate after errors.

Permanent:
- 404/410;
- invalid URL.

Do not retry forever.

## 10. Recrawl

Pages change at different rates.

Store metadata:
```text
last_crawled
last_changed
etag
last_modified
change_frequency estimate
priority
```

Next crawl time can adapt:
- frequently changing news → minutes;
- static documentation → days/weeks.

Conditional HTTP:
```text
If-None-Match
If-Modified-Since
```
can save bandwidth.

## 11. News Freshness Change

Need high news freshness **without violating politeness**.

Increase scheduling priority for:
- trusted news domains;
- known feeds/sitemaps;
- pages with high historical change frequency.

But enforce:
```text
per-host max concurrency/rate
```
even for high priority.

Priority determines *which eligible host/page goes next*, not permission to overload host.

Separate global priority and host politeness constraints.

## 12. Frontier Durability

If scheduler crashes, do not lose millions of pending URLs.

Store frontier durably:
- partitioned queues/log;
- persistent scheduler state.

Worker fetch can be at-least-once.
Duplicate fetch is acceptable if dedup/idempotent storage exists.

## 13. Partitioning

Natural host-based component:

```text
hash(host) → scheduler shard
```

Keeps one host's rate state in one shard.

Problem:
one giant host (e.g. huge site) becomes hot.

Can subdivide pages but keep centralized host token/rate authority.

## 14. Backpressure

If parsing/indexing slower than fetch:
- stop/slow fetch scheduling;
- bounded queues;
- avoid unlimited content backlog.

If storage unavailable:
- pause fetch rather than download data you cannot persist.

## 15. Storage

- raw page/content → object/blob store;
- URL metadata → DB/KV;
- link graph → graph/adjacency storage if needed;
- search indexing → downstream derived search system.

Crawler itself does not need one giant relational schema.

## 16. Failure Scenarios

### Worker dies after fetch
URL may be retried; content write idempotent by URL/version/hash.

### Scheduler shard dies
Replica/failover; durable frontier.

### Robots fetch fails
Conservative policy depending product; avoid hammering.

### One domain returns 429
Back off host, honor retry guidance.

### Parser bug creates infinite URL variants
Canonicalization + per-domain quotas + frontier growth alerts.

## 17. Security

Fetching arbitrary URLs introduces risks:
- SSRF into private/internal addresses;
- huge response bodies;
- decompression bombs;
- malicious content;
- redirect loops.

Crawler workers should:
- block private/internal network ranges;
- cap size/time;
- sandbox parsers;
- limit redirects.

## Interview Questions

1. What is URL frontier?
2. Why global FIFO is unsafe?
3. How do you enforce politeness?
4. URL dedup vs content dedup?
5. How schedule recrawl?
6. How prioritize news safely?
7. What happens if downstream indexing is slow?
8. How do you avoid SSRF?
9. What is a reasonable shard key?

## Common Mistakes

- Queue only, no host politeness.
- Workers independently decide rate with no shared host state.
- No canonicalization.
- Retry 404 forever.
- High-priority news bypasses rate limits.
- Frontier not durable.
- No backpressure.
- Crawler allowed to fetch internal/private IPs.
- All content in metadata DB.

## Must Remember

- **Frontier scheduling is the heart of crawler design.**
- **Canonicalize before dedup.**
- **URL dedup and content dedup solve different problems.**
- **Politeness is per host/domain and must survive many workers.**
- **Priority does not override host safety.**
- **Recrawl frequency follows change/freshness value.**
- **Frontier must be durable and partitioned.**
- **Backpressure prevents fetch from outrunning storage/indexing.**
- **Crawler workers need SSRF/resource protections.**

## Self-Score

Use the 40-point rubric. Redo scheduler/frontier if your first design was only “Kafka + workers.”
