# Session 45 — Mock #5 — Correctness / Concurrency / Commerce

**Phase:** Phase 4 — Timed Mock Interviews  
**Recommended time:** 45–55 minute mock + 30–45 minute review

## Session Goal

Test whether your business invariant survives duplicate requests, races, timeouts, and partial failure.

## What You Need to Read / Learn

- Choose an unseen correctness prompt such as hotel reservation, digital wallet, checkout, coupon redemption, or payment workflow.
- Write the core business invariant before architecture.
- Inject a duplicate request, race, and dependency timeout.
- Use database atomicity/constraints/versioning/locks intentionally.
- If multiple services participate, include idempotency, saga/outbox/reconciliation as needed.

## What You Need to Do

- [ ] Ask the interviewer to introduce a timeout after the external side effect may have happened.
- [ ] Walk the state machine and every failure path.
- [ ] Redo the correctness section until no invariant violation remains.

## **Must Remember for the Interview**

- **Correctness should be expressed as an invariant and state transitions.**
- **Timeout means uncertain state, not safe failure.**
- **Idempotency protects retries; transactions/locking protect concurrency; saga/reconciliation protect multi-service workflows.**
- **Prefer local atomic mechanisms before distributed locks.**
- **A system that scales but violates money/inventory invariants is not a passing design.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Invariant first.**
- **Model states/transitions.**
- **Handle duplicate + race + timeout explicitly.**
- **Use idempotency and reconciliation for uncertain side effects.**
- **Pass target = 32/40 with Correctness ≥2; aim for ≥3.**

## Self-Test Before Marking This Session Complete

- [ ] Did I state the invariant?
- [ ] Did I prevent concurrent double allocation/charge?
- [ ] Did I handle timeout after an unknown outcome?
- [ ] Did I use idempotency correctly?
- [ ] Did I include reconciliation?

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

**Progress:** Session 45/46  
**Next:** Session 46
