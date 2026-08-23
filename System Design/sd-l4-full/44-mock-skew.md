# Lesson 44 — Mock #4 — High Scale with Skew

**Phase:** Timed Mock  
**Session:** 44/46  
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

> **Design Trending Topics**

Compute and serve frequently refreshed trending topics from a high-volume stream of searches or posts. Support regional views. Clarify the trend window, freshness target, abuse resistance, and whether exact counts are required.

Do not read the next sections until your first architecture is coherent.

## Mid-interview change

> At minute ~25: a celebrity event causes extreme concentration around a few keys.

Adapt the existing design. Do not restart.

## Failure scenario

> At minute ~40: one stream partition becomes hot and consumer lag grows rapidly.

Explain user impact, correctness impact, detection, immediate behavior, recovery, and longer-term improvement.

## What this mock is testing

This prompt tests whether you can design an **aggregation system under skew**. The important word is not “trending”; it is **hot keys and approximate/time-window computation**.

## Before the timer: mental checklist

Clarify:

```text
trend window (5 min? 1 hour?)
refresh frequency
regional/global
exact counts required?
source event volume
abuse/spam filtering
number of candidate topics
```

Exact global ranking every second may be unnecessary. Product semantics determine how approximate the pipeline can be.

## What strong performance looks like

### Event ingestion

You use a durable stream/aggregation path at large scale and define event partitioning. You separate raw signals from served top-K results.

### Windowing

You explain how counts decay/expire—time buckets, sliding windows, or stream-processing windows. You do not let “all-time count” accidentally become “trending.”

### Top-K

You can compute local/partition top-K then merge, reducing centralized load. Exact algorithms are not required, but you should understand that global aggregation can become a bottleneck.

### Celebrity skew change

A few keys can dominate one partition. You may salt/split hot-key counts across subkeys, aggregate locally, then merge. This trades exact immediate value for scalable write distribution.

### Consumer lag failure

If one partition lags, trends become stale/incomplete. Monitor event-time lag, rebalance/split hot partitions, and keep serving last known results rather than failing the read API.

## Interviewer-style follow-up prompts

- What defines a trend mathematically at a high level?
- How do old events stop contributing?
- How do you detect/manipulate bot spam?
- Can you tolerate approximate counts?
- What happens when one topic has 50% of all events?
- How do regional trends differ from global?
- How do you serve top 100 cheaply?
- How do you rebuild after an aggregation bug?
- What data do you retain for replay?
- What if update pipeline is five minutes behind?

## Common failure patterns to watch

- single global counter per topic with no hot-key plan;
- one central reducer for all traffic;
- no time-window expiration;
- served API reads raw stream on request;
- no spam/abuse thought;
- consumer lag treated as binary up/down;
- approximate result rejected without business reason.

## Revision topics before attempting

- [ ] streaming partitions
- [ ] aggregation windows
- [ ] top-K
- [ ] sharded counters
- [ ] hot keys
- [ ] event-time lag
- [ ] replay
- [ ] regional partitioning

## After the mock: short reflection

```text
Did my design truly compute “trending,” not popularity?
Where did aggregation happen?
What was hot-key mitigation?
Could I serve stale last-known trends during lag?
Was exactness justified?
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

**Next:** Lesson 45 — Mock #5 — Correctness / Concurrency
