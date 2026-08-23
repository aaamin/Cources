# Lesson 36 — Design a Distributed Web Crawler

**Phase:** Guided Design  
**Session:** 36/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Discover and fetch pages.
- Respect robots and per-host politeness.
- Deduplicate URLs/content.
- Prioritize important/fresh pages.
- Recover cleanly from worker crashes.

## 2. Scale and workload shape

Estimate pages/day, average page bytes, number of hosts, desired recrawl intervals, outbound bandwidth, and storage. Throughput is constrained not only by your cluster but by **politeness toward external sites**.

## 3. API / contract surface

This is mainly an internal pipeline. Core records/events are:

```text
DiscoveredURL
FetchTask
FetchResult
ParsedLinks
```

Operational APIs may exist for seeding, pausing domains, or inspecting frontier state.

## 4. Data model

```text
Frontier(url, host, priority, next_fetch_at, lease?)
SeenURL(canonical_hash, first_seen, last_seen)
Page(url, content_ref, fetched_at, status, content_hash)
HostPolicy(host, robots, crawl_delay, next_allowed_at)
```


## 5. High-level architecture

```text
Seed URLs
   ↓
URL Frontier / Host Scheduler
   ↓
Fetch Workers
   ↓
Parser
  ├─ Content/Object Store
  └─ Extracted URLs → Canonicalizer/Deduper → Frontier
```

The scheduler, not individual workers, should enforce host-level rate limits consistently.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### URL canonicalization and dedupe

Normalize equivalent URLs before seen-checks: case rules, default ports, fragments, tracking parameters according to policy. Content hashes can detect identical pages reachable through different URLs.

### Politeness

Maintain per-host concurrency and next-allowed-fetch time. A global high-throughput queue without host scheduling can accidentally attack one small site.

### Priority and recrawl

Use importance/freshness signals to rank the frontier. News homepages may recrawl in minutes; static archives in weeks. Store next-fetch time and priority rather than treating all URLs equally.

### Leases

Workers lease fetch tasks. If a worker dies, the lease expires and another worker retries.

## 7. Failure modes and recovery

- Worker crash: task lease expires; retry with bounded attempts.
- Poison page/parser bug: isolate process, retry limit/DLQ.
- Duplicate URLs: canonical hash dedupe before enqueue.
- Frontier scheduler crash: durable frontier/checkpoint.
- DNS/host failure: backoff per host rather than repeatedly retrying.
- Sudden news priority: increase crawl frequency without breaking per-host limits.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Maximum fetch throughput is not the only goal. A good crawler balances freshness, coverage, politeness, fairness, deduplication cost, and storage.

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

### Frontier is the heart of the crawler

The frontier is not merely a FIFO queue. It decides **what to crawl next** while respecting priority, recrawl deadlines, and host politeness. A scalable design often groups URLs by host and schedules hosts fairly so one prolific domain cannot dominate all workers.

### Canonicalization is policy

`http://example.com`, `https://example.com/`, URLs with tracking parameters, fragments, or alternate encodings may or may not represent the same resource. There is no universal canonicalizer. Apply safe normalization and use site-declared canonical metadata when appropriate; overly aggressive normalization can merge distinct pages.

### Robots and ethics are part of correctness

Fetch robots policies, cache them with expiry, respect disallow rules and crawl delays, identify the crawler, and provide operator controls. “Scale” is not permission to ignore external-system constraints.

### Fetch vs parse isolation

Fetching is network-bound and interacts with hostile/unreliable content. Parsing may be CPU/memory intensive and should run in isolated workers/sandboxes. Separating stages prevents a pathological page from taking down the fetch fleet.

### Recrawl scheduling

Track change frequency. Pages that change often should recrawl more frequently; static pages can back off. HTTP validators such as ETag/Last-Modified can reduce bandwidth when content is unchanged.

### Common interview mistakes

- One global FIFO that hammers one host.
- No durable frontier, so restart loses crawl progress.
- Treating DNS/connection errors as immediate tight-loop retries.
- Deduplicating only after download, wasting bandwidth.
- Assuming every page deserves the same recrawl interval.
- Forgetting that parsing arbitrary web content is a security risk.

### Reusable patterns learned

Fair scheduling, per-key rate limits, leases, dedupe, recursive discovery, incremental recrawl, external-system politeness, and durable work queues.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 37 — Design Uber / Ride Matching
