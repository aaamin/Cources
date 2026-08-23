# Lesson 45 — Mock #5 — Correctness / Concurrency

**Phase:** Timed Mock  
**Session:** 45/46  
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

> **Design Hotel Reservation**

Users search room inventory, place temporary holds, and pay to confirm reservations. The system must not confirm the same inventory twice for overlapping dates. Clarify inventory granularity, hold duration, cancellation policy, and payment semantics.

Do not read the next sections until your first architecture is coherent.

## Mid-interview change

> At minute ~25: the payment provider becomes slow and clients retry requests.

Adapt the existing design. Do not restart.

## Failure scenario

> At minute ~40: the reservation service crashes after payment succeeds but before confirmation is persisted.

Explain user impact, correctness impact, detection, immediate behavior, recovery, and longer-term improvement.

## What this mock is testing

This is a **business-invariant and uncertain-outcome** interview. Scale matters, but correctness under concurrency, timeout, and retry is the central signal.

> Write the invariant before drawing architecture.

Example form:

```text
For a given hotel/room inventory/date, confirmed reservations cannot exceed available inventory.
```

## Before the timer: mental checklist

Clarify:

```text
room-specific vs room-type inventory
stay date ranges
temporary hold duration
cancellation/refund
payment authorization/capture
regional ownership
search freshness
```

Search availability can be approximate. Final hold/confirmation cannot.

## What strong performance looks like

### Inventory model

You identify the authoritative inventory unit and use transaction/conditional updates to prevent oversell. If inventory is counts by room type/date, the invariant spans each date in requested range and needs atomic/serialized reservation semantics.

### Hold state

You use expiring holds so users can pay without indefinitely blocking inventory. Expiration is enforced in authoritative logic, not solely by a cleanup worker.

### Payment timeout change

You treat timeout as **unknown outcome**. Retry by same idempotency key or query payment status. You do not charge again blindly.

### Crash after payment failure scenario

You have durable workflow state and reconciliation. A successful payment with pending reservation can be discovered and resolved according to business policy—confirm if inventory is still owned/held, otherwise refund/compensate.

### Search vs booking

Browse/search can be cached/eventually consistent; final reservation validates current source of truth. This keeps the critical correctness path narrow.

## Interviewer-style follow-up prompts

- How do you prevent two users taking the last room?
- Does one DB transaction cover multiple stay dates?
- What if a hold expires during payment?
- What if refund fails?
- How do retries avoid duplicate reservation?
- Can two regions confirm the same inventory?
- How does cancellation return inventory?
- Is a distributed lock necessary?
- What is periodically reconciled?
- What if search says available but booking says sold out?

## Common failure patterns to watch

- check-then-write race;
- cache authoritative for inventory;
- payment timeout treated as failure;
- no idempotency key;
- Saga with no reconciliation;
- hold expiration worker required for correctness;
- active-active inventory across regions with no coordination;
- “exactly once” claim with no mechanism.

## Revision topics before attempting

- [ ] invariant/state machine
- [ ] transactions/conditional writes
- [ ] optimistic/pessimistic locking
- [ ] idempotency keys
- [ ] hold lease/expiry
- [ ] Saga/compensation
- [ ] outbox/reconciliation
- [ ] consistency under partition

## After the mock: short reflection

```text
Did I state the invariant clearly?
Where exactly is it enforced?
Did I model unknown payment outcome?
Can every retry be safe?
What repairs partial failure?
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

**Next:** Lesson 46 — Final Unseen FAANG-Style Mock
