# Lesson 42 — Mock #2 — Realtime / Async Workflow

**Phase:** Timed Mock  
**Session:** 42/46  
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

> **Design Live Sports Comments**

Users watching a major sports event can post comments and receive new comments with low latency. A single event can attract millions of viewers at once. Clarify ordering scope, moderation, retention, and whether every viewer must receive every comment.

Do not read the next sections until your first architecture is coherent.

## Mid-interview change

> At minute ~25: add moderation and per-event rate limits without making the posting path unusably slow.

Adapt the existing design. Do not restart.

## Failure scenario

> At minute ~40: a delivery worker repeatedly crashes and some comment events are retried.

Explain user impact, correctness impact, detection, immediate behavior, recovery, and longer-term improvement.

## What this mock is testing

This prompt stresses **realtime delivery + burst traffic + asynchronous moderation/fan-out**. It is not enough to say “WebSockets + Kafka.” You need to define the delivery semantics and what happens when millions of viewers cluster around one event.

## Before the timer: mental checklist

Clarify:

```text
Does every viewer see every comment?
Ordering: global, per event, approximate?
Retention/history?
Posting rate vs viewing rate?
Moderation before or after visibility?
Latency target?
Anonymous/authenticated?
```

If every one of 5M viewers receives every comment, fan-out may be impossible/unnecessary. Product requirements may allow sampling/ranking/top comments.

## What strong performance looks like

### Realtime connection reasoning

You choose a mechanism deliberately. If WebSockets/SSE are used, you explain gateway ownership, reconnect behavior, and how events reach the correct viewers.

### Event partitioning

You define ordering scope. An event ID/topic partition by `event_id` may preserve event-local order but a mega-event becomes a hot partition. You recognize this and discuss whether exact total order is truly needed.

### Fan-out

You avoid writing one durable copy per viewer. Comments are stored once; gateways/subscribers receive event stream/batches. Extremely large audiences may need hierarchical fan-out or edge/pub-sub layers.

### Moderation change

When moderation is introduced, state whether comments are pre-moderated (higher latency) or appear optimistically and may be removed. Critical abuse filters can be synchronous; expensive ML can be async depending on product safety requirement.

### Retry failure

If worker retries a comment event, downstream delivery/storage should dedupe by stable comment/event ID. At-least-once processing should not create duplicate visible comments.

## Interviewer-style follow-up prompts

- What happens when one event gets 100× the normal audience?
- Do you need strict ordering of comments?
- How would clients reconnect after 30 seconds offline?
- What if one gateway goes down with 100k sockets?
- How do you prevent one user from spamming?
- How do you apply moderation without blocking the entire stream?
- Do you store every comment forever?
- How would you show only “top” comments to viewers?
- What metric reveals realtime delivery falling behind?
- What if the event ends and millions of sockets disconnect/reconnect elsewhere?

## Common failure patterns to watch

- one global ordered stream for every comment;
- fan-out one message into millions of durable per-user rows;
- no connection reconnect/cursor story;
- WebSocket treated as durable state;
- retry duplicates visible comments;
- moderation/rate limiting bolted on with no latency discussion;
- no hot-event partition strategy.

## Revision topics before attempting

- [ ] WebSockets/SSE
- [ ] connection gateway
- [ ] Pub/Sub/stream partitions
- [ ] ordering scope
- [ ] at-least-once + dedupe
- [ ] backpressure
- [ ] rate limiting
- [ ] hot partitions
- [ ] async moderation trade-off

## After the mock: short reflection

```text
Did I define ordering scope?
Did I distinguish storage from delivery?
Did I handle reconnect?
Did I handle hot-event fan-out?
Did moderation change my architecture cleanly?
Did retry create duplicate effects?
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

**Next:** Lesson 43 — Mock #3 — Data-Heavy / Write-Heavy
