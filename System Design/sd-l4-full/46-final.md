# Lesson 46 — Final Unseen FAANG-Style Mock

**Phase:** Timed Mock  
**Session:** 46/46  
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

> **Design a Global E-Commerce Checkout System**

Users place orders, reserve inventory, pay, and receive confirmation. The service operates in multiple regions. Clarify product inventory ownership, latency goals, consistency needs, region strategy, and what “checkout complete” guarantees.

Do not read the next sections until your first architecture is coherent.

## Mid-interview change

> At minute ~25: require regional failover while preserving order correctness.

Adapt the existing design. Do not restart.

## Failure scenario

> At minute ~40: one region loses connectivity to another during a major sales event.

Explain user impact, correctness impact, detection, immediate behavior, recovery, and longer-term improvement.

## What this final mock is testing

This mock deliberately combines many course themes: **data modeling, transactions, distributed workflow, idempotency, inventory contention, payment ambiguity, multi-region routing, failover, and graceful degradation**.

It is not expected that every subsystem is deeply designed. The final test is whether you can **prioritize the hardest correctness requirement and control the interview**.

## Before the timer: final mental checklist

Ask only high-value questions:

```text
What does checkout include?
When is an order considered confirmed?
Inventory ownership model?
Payment semantics?
Latency/availability priority?
Can regions accept writes independently?
RPO/RTO?
```

Choose non-goals such as product search, recommendation, and warehouse fulfillment unless the interviewer brings them in.

## What strong performance looks like

### Scope control

You focus on:

```text
cart/order submission
inventory reservation
payment
order confirmation
```

and keep unrelated e-commerce systems out.

### Workflow correctness

You state invariants such as:

- no duplicate order from client retry;
- inventory not oversold;
- payment not double charged;
- confirmed order eventually reconciles to payment/inventory state.

You use local transactions where possible and Saga/outbox/reconciliation across services.

### Data ownership

Order Service owns order state. Inventory owns stock/reservations. Payment owns payment attempts/results. Services communicate through APIs/events rather than sharing mutable tables.

### Multi-region change

You do not immediately promise global active-active writes for the same inventory. A strong approach may assign inventory/order authority to a home region, route writes accordingly, and use asynchronous replicas elsewhere. If the requirement truly demands active-active, you explain the coordination/consistency cost.

### Partition failure scenario

When regions cannot communicate, you explicitly decide which operations remain available and which reject/delay to protect invariants. This is the practical consistency/availability trade-off learned earlier.

### Production thinking

You mention:

- idempotency keys;
- outbox/events;
- retry/backoff;
- reconciliation;
- metrics/SLO;
- audit/security around payments;
- overload/admission control for major sale.

But you do not turn the final five minutes into a checklist recital; connect each to a real risk.

## Final interviewer-style follow-ups

- What happens if inventory succeeds and payment fails?
- What happens if payment succeeds and Order Service crashes?
- How do you retry checkout safely?
- Which store is authoritative for order state?
- How is `OrderConfirmed` published reliably?
- What if the notification system is down?
- What if one product becomes a flash-sale hotspot?
- How do you prevent retry storms during payment outage?
- What happens during regional network partition?
- Which data needs strong consistency and which can lag?
- What is your first 10× bottleneck?
- What would you build in V1 vs later?

## Common final-mock failure patterns

- enormous architecture before requirements;
- no explicit invariant;
- “microservices + Kafka” with no transaction story;
- payment timeout handled incorrectly;
- inventory cache used for final correctness;
- active-active region claim with no conflict/coordination plan;
- no recovery/reconciliation;
- running out of time before failures/trade-offs;
- silent drawing instead of communication.

## Final revision topics

Before this mock, you should be comfortable with:

- [ ] requirements/non-goals
- [ ] estimation
- [ ] API + data modeling
- [ ] SQL transactions/indexes
- [ ] replication/sharding
- [ ] consistency/CAP
- [ ] queues/streams
- [ ] idempotency
- [ ] Saga/outbox/reconciliation
- [ ] reliability/backpressure
- [ ] multi-region/RPO/RTO
- [ ] observability/security
- [ ] 2-minute closing summary

## Final self-review prompts

```text
Did I lead the interview or react to it?
Did every major component solve a stated requirement?
Did I separate source of truth from derived data?
Did I explain uncertain outcomes and retry behavior?
Did I protect business invariants during region failure?
Did I discuss one credible alternative?
Did I finish with a clear summary?
```

> **Course readiness is repeatability, not one good score.** If this mock is below the gate, repair the weakest category and perform another unseen prompt rather than restarting the curriculum.

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

**Course complete:** evaluate the readiness gate in `README.md`.
