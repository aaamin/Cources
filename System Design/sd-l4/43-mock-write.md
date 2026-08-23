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
