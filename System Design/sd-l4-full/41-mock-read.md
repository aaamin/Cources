# Lesson 41 — Mock #1 — Read-Heavy Service

**Phase:** Timed Mock  
**Session:** 41/46  
**Recommended time:** 45–55 minute mock + 30–45 minute review

## Purpose

There is deliberately **no reference solution** in this lesson. The goal is to prove transfer to an unseen problem rather than memorize another diagram.

## How to run it

Use a blank diagram and a timer. Speak while designing. If possible, use a peer interviewer; otherwise record yourself.

```text
0–6 min   requirements + non-goals
6–10      workload / estimates
10–16     API + data model
16–28     HLD + flows
28–42     deep dive
42–49     failures / production review
49–52     summary
```

## Prompt

> **Design a Product Catalog and Filtering Service**

Users browse a very large e-commerce catalog, filter by attributes, sort results, and open product detail pages. Product updates are much less frequent than reads. Clarify whether search is exact/filter-based or full text, what freshness is expected, and whether inventory belongs in scope.

Do not read the next sections until your first architecture is coherent.

## Mid-interview change

> At minute ~25: inventory badges must become near-real-time without making the whole catalog strongly consistent.

Adapt the existing design. Do not restart.

## Failure scenario

> At minute ~40: the distributed cache becomes unavailable during peak traffic.

Explain user impact, correctness impact, detection, immediate behavior, recovery, and longer-term improvement.

## What this mock is testing

This prompt is deliberately **read-heavy and discovery-oriented**. A good interview should reveal whether you can keep several concerns separate:

- catalog/product data as source of truth;
- filtering/search indexes as derived serving structures;
- cache/CDN for hot reads;
- near-real-time inventory as data with different freshness/correctness needs;
- graceful behavior when the cache disappears.

Do not memorize a particular product-catalog architecture before the mock. Instead, demonstrate that you can discover these requirements through reasoning.

## Before the timer: mental checklist

Do not write a solution, but remember to clarify:

```text
How many products?
How many attributes/categories?
Read/write ratio?
Full text vs filters?
Inventory/price freshness?
Personalization?
Regional traffic?
```

The interviewer may intentionally leave “inventory badges” ambiguous. Ask whether a stale badge is acceptable if final checkout validates authoritative inventory.

## What strong performance looks like

### Requirements

You keep the scope under control. You distinguish browse/search from transaction/checkout and state what is not being designed.

### Estimation

You notice the workload is read-heavy and estimate catalog size, query QPS, update rate, and perhaps response/cache size. You do not waste time estimating irrelevant media storage unless images become part of the prompt.

### Data modeling

You identify product identity, attributes/category, price, and perhaps inventory reference. You state which store is authoritative and which structures are indexes/caches.

### Architecture

You start simple enough that every component is justified. If you introduce a search index, explain how updates reach it and what lag means. If you introduce cache, explain key/TTL/invalidation and cache-outage behavior.

### Requirement change

When near-real-time inventory is added, you do not make every catalog document synchronously strongly consistent. You reason about separating slowly changing product metadata from fast-changing inventory and validating authoritative availability at the right step.

### Failure scenario

When the cache fails, you recognize that “fall back to DB” can overload the source. You discuss load protection, degraded behavior, gradual warmup, or read replicas rather than treating fallback as free.

## Interviewer-style follow-up prompts

Use these only after your first design:

- What is the source of truth for product metadata?
- What is the source of truth for inventory?
- How does a product update reach the search/filter index?
- How stale can search results be?
- What happens if one product is viewed 1M times/minute?
- How do you paginate stable filtered results?
- What breaks first at 10× read traffic?
- How would you support 100M products and thousands of sellers?
- What if a search-index rebuild produces bad results?
- How would you keep browse available during a database failover?

## Common failure patterns to watch in your own answer

- treating search index as authoritative product database;
- “use Redis” with no cache policy/fallback;
- synchronously joining inventory for every search result when approximate badges would do;
- no pagination/indexing plan;
- no update path from source data to derived search index;
- no hot-product strategy;
- overengineering checkout/payment even though prompt is browse/catalog.

## Revision topics before attempting

If any of these feel weak, revise the lesson before starting the timer:

- [ ] caching + cache outage
- [ ] SQL/indexing vs search index
- [ ] source of truth vs derived view
- [ ] cursor pagination
- [ ] read replicas
- [ ] eventual consistency/freshness
- [ ] hot keys
- [ ] graceful degradation

## After the mock: short reflection

Write one sentence for each:

```text
My strongest decision:
My weakest assumption:
The first bottleneck at 10×:
One unnecessary component I added:
One missing failure case:
One trade-off I explained well:
```

## Important: how to use this mock

> **Important:** This file intentionally does **not** contain a reference architecture. Its job is to test transfer. If you study a canonical solution before the timer, the mock loses most of its value.

For a self-run mock, record your voice or screen. During review, do not grade yourself on whether your boxes match somebody else's diagram. Grade whether the design follows from requirements, protects its important invariants, has a complete request flow, and survives the injected failure.

### A useful self-interviewer technique

At minute 20–25, stop and ask yourself:

```text
What requirement have I not addressed?
What is the source of truth?
What is the first bottleneck at 10×?
What happens if the most important dependency becomes slow?
```

At minute 40, inject the provided failure even if your architecture is unfinished. Real interviews often redirect you before you feel ready. The ability to adapt is part of the signal.

### How to review without memorizing

After the attempt, compare your decisions to principles from Lessons 1–40 rather than searching immediately for “the solution.” For every weak section write:

```text
Requirement I missed:
Principle I should have applied:
Specific design change:
Trade-off introduced by that change:
```

Only after this repair should you consult external/reference designs, and then use them to discover missing patterns—not to replace your own reasoning.

### Score interpretation discipline

A total score can hide a fatal weakness. A 34/40 with 1/4 in correctness on a booking system is not a good mock. Read the category scores first, total second. Repeated weakness in one category should send you back to the corresponding fundamental lesson for narrow repair.

## 40-point rubric

| Category | 0 | 2 | 4 |
|---|---|---|---|
| Requirements & scope | Missing | Main features only | Priorities, constraints, non-goals |
| Estimation & workload | Missing/decorative | Reasonable | Drives design; peak/skew included |
| API/events/data | Missing | Happy path | Access-pattern driven; errors/idempotency |
| HLD & flows | Incomplete | Coherent | Simple, complete, narrated |
| Scalability | Ignored | Generic | Workload-specific bottlenecks/10× path |
| Correctness | Ignored | Names consistency | Invariants, races, duplicates, recovery |
| Reliability/ops | Ignored | Basic redundancy | Degradation, monitoring, deploy/recovery |
| Security/privacy/cost | Ignored | Mentions basics | Relevant risks/costs prioritized |
| Trade-offs/evolution | None | One alternative | Credible alternatives and evolution |
| Communication/time | Unclear | Understandable | Leads, adapts, finishes, summarizes |

A strong readiness signal is **32/40 or higher** with no category below 2. A 0 or 1 in requirements, API/data, HLD, correctness, trade-offs, or communication is a non-pass regardless of total.

## Review process

1. Score all ten categories.
2. Identify the bottom two.
3. Write three concrete misses.
4. Redo only the weakest 15–20 minutes.
5. Perform narrow drills for the weak categories.
6. Use a new unseen prompt for the next full attempt.

## Score sheet

```text
Requirements & scope:       /4
Estimation & workload:      /4
API / events / data:        /4
HLD & flows:                /4
Scalability:                /4
Correctness:                /4
Reliability / operations:   /4
Security / privacy / cost:  /4
Trade-offs / evolution:     /4
Communication / time:       /4

TOTAL:                      /40
```

## Repair notes

```text
Weak category #1:
Weak category #2:
Three biggest misses:
1.
2.
3.
What I changed after review:
What I will test next:
```

---

**Next:** Lesson 42 — Mock #2 — Realtime / Async Workflow
