# Session 42 — Mock #2 — Realtime / Messaging / Async Workflow

**Phase:** Phase 4 — Timed Mock Interviews  
**Recommended time:** 45–55 minute mock + 30–45 minute review

## Session Goal

Test whether realtime connections or async workflows cause you to lose structure or correctness under time pressure.

## What You Need to Read / Learn

- Use an unseen realtime/collaboration prompt such as live comments, multiplayer presence, calendar reminders, or live auction.
- Do not reuse WhatsApp unchanged.
- Force yourself to separate transport from durable state and delivery semantics.
- Include at least one requirement change after the HLD.
- Review queue/retry/idempotency or connection/reconnect semantics only after the attempt.

## What You Need to Do

- [ ] Ask the interviewer/peer to introduce a requirement change around minute 25–30.
- [ ] Explicitly define ordering and delivery semantics.
- [ ] After the mock, redo only the realtime/async deep dive if it was weak.

## **Must Remember for the Interview**

- **Realtime transport does not replace durability, ordering, or idempotency design.**
- **Requirement changes should modify part of the design, not make you restart from zero.**
- **Define online/offline/reconnect behavior when relevant.**
- **If async, discuss retries, duplicates, and queue backlog.**
- **Protect time for production/failure review.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Choose polling/SSE/WebSocket based on direction/frequency.**
- **Define ordering scope and durable source of truth.**
- **Async pipelines require retry/idempotency/backpressure.**
- **Pass target = 32/40 with no category below 2.**
- **Keep the interview structure even after a midstream change.**

## Self-Test Before Marking This Session Complete

- [ ] Did I distinguish transport from persistence?
- [ ] Did I handle reconnect/retry/duplicates?
- [ ] Did I respond calmly to the requirement change?
- [ ] Did I leave time for failures/trade-offs?
- [ ] What two categories need repair?

## Completion Rule

Mark this session complete only after the timed mock, rubric score, and targeted repair notes. **Do not use the total score to hide a critical category weakness.**


## Session-Specific Notes

### 40-Point Rubric

Score 0–4 in each category:

1. Requirements & Scope
2. Estimation & Workload
3. APIs / Events / Data Model
4. High-Level Design & Flows
5. Scalability & Performance
6. Correctness & Consistency
7. Reliability & Operations
8. Security / Privacy / Cost
9. Trade-Offs & Evolution
10. Communication & Time Control

**Target:** 32/40 or higher, with no category below 2. A 0–1 in Requirements, APIs/Data, HLD, Correctness, Trade-offs, or Communication is a mock failure regardless of total score.


---

**Progress:** Session 42/46  
**Next:** Session 43
