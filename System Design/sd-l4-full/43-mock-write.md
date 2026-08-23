# Lesson 43 — Mock #3 — Data-Heavy / Write-Heavy

**Phase:** Timed Mock  
**Session:** 43/46  
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

> **Design an IoT Telemetry Platform**

Millions of devices send measurements continuously. Users query recent readings and hourly aggregates. Data has tiered retention. Clarify device authentication, ordering needs, tolerated ingest delay, and query dimensions.

Do not read the next sections until your first architecture is coherent.

## Mid-interview change

> At minute ~25: support replay of historical events into a new aggregation job without stopping ingestion.

Adapt the existing design. Do not restart.

## Failure scenario

> At minute ~40: one tenant begins producing 20× its normal traffic.

Explain user impact, correctness impact, detection, immediate behavior, recovery, and longer-term improvement.

## What this mock is testing

This prompt is a **write-heavy ingestion + streaming + retention + replay** test. The system should keep accepting telemetry even when query/index components are degraded.

## Before the timer: mental checklist

Clarify:

```text
devices count / events per sec
payload size
ordering per device?
acceptable ingest loss?
query dimensions?
recent vs historical latency?
retention periods?
tenant isolation?
```

The dominant scale is sustained writes and storage, not frontend read QPS.

## What strong performance looks like

### Ingestion

You batch where appropriate, authenticate devices/agents, and place a durable buffering/stream layer before expensive downstream processing when scale justifies it.

### Partitioning

You choose a key that spreads tenants/devices while preserving needed per-device/series order. You explicitly discuss a large tenant/hot partition.

### Storage tiers

You separate raw durable events, recent query-optimized time-series data, and aggregated historical rollups. You do not index every byte forever in an expensive search system.

### Replay change

When historical replay is requested, your original architecture should make it possible from retained stream/raw object storage. You run the new aggregation as a separate consumer/version so normal ingestion continues.

### Noisy tenant failure

One tenant at 20× should be rate-limited/isolated, not allowed to exhaust shared partitions and make every tenant lose data.

## Interviewer-style follow-up prompts

- Can events arrive late or out of order?
- How do you identify duplicates?
- What happens when the stream consumer is hours behind?
- How do you query one year of data cheaply?
- What if raw retention is seven years?
- Which metrics would you alert on?
- How do you enforce tenant quotas?
- How do you rebuild a bad aggregation?
- What if devices are offline for a day then reconnect?
- What is the source of truth for replay?

## Common failure patterns to watch

- direct write from every device into one relational table;
- query database blocks ingestion;
- no durable raw/replay source;
- high-cardinality dimensions ignored;
- one noisy tenant dominates cluster;
- consumer lag not monitored;
- storing full-resolution data forever in hot storage.

## Revision topics before attempting

- [ ] batching
- [ ] queues/streams
- [ ] partition keys
- [ ] consumer lag
- [ ] time-series/rollups
- [ ] retention tiers
- [ ] replay/idempotency
- [ ] quotas/backpressure

## After the mock: short reflection

```text
What was my durable ingest boundary?
Could query failure lose data?
Could I replay?
How did I isolate a hot tenant?
What was my retention/cost story?
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

**Next:** Lesson 44 — Mock #4 — High Scale with Skew
