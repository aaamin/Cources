# Session 22 — Idempotency, Transactions, Concurrency & Distributed Workflows

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn how to preserve business invariants when requests retry, workers race, and multi-service workflows partially fail.

## What You Need to Read / Learn

- Race conditions and lost updates.
- Optimistic concurrency control/version checking.
- Pessimistic locking.
- Compare-and-set.
- Unique constraints as correctness tools.
- Leases and distributed locks at conceptual depth; stale-owner problem and fencing-token intuition.
- Idempotency keys and deduplication records.
- Exactly-once business effect versus transport delivery claims.
- Saga pattern and compensating actions.
- Transactional outbox for avoiding database+message dual writes.
- Change Data Capture (CDC) at recognition depth.
- Reconciliation jobs for repairing uncertain/partial outcomes.
- CQRS/event sourcing: recognition-level benefits/costs, not implementation depth.

## What You Need to Do

- [ ] Design a payment endpoint safe against client retries.
- [ ] Design a ticket hold state transition safe against two simultaneous buyers.
- [ ] Draw order → inventory → payment workflow and show timeout/compensation paths.
- [ ] Explain the dual-write bug when writing DB state and publishing an event separately; repair with an outbox.

## **Must Remember for the Interview**

- **Idempotency means repeating the same logical request does not create an additional business effect.**
- **A timeout can leave the operation in an unknown state; retry design must account for this.**
- **Use database constraints/atomic operations before reaching for distributed locks.**
- **Saga provides recovery/compensation, not distributed ACID atomicity.**
- **Transactional outbox solves the 'DB committed but event publish failed' class of dual-write problem.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Protect invariants explicitly: no double charge, no duplicate seat sale, no lost update.**
- **Idempotency key + stored result/status is a common request-retry pattern.**
- **Optimistic locking fits low-conflict updates; pessimistic locking fits stronger contention needs.**
- **Saga = sequence of local transactions + compensation/recovery.**
- **Outbox + CDC/publisher gives durable state-to-event propagation.**
- **Reconciliation is essential for uncertain distributed workflows.**

## Self-Test Before Marking This Session Complete

- [ ] Can I design an idempotent payment API?
- [ ] Can I explain optimistic vs pessimistic locking?
- [ ] Can I explain saga vs transaction?
- [ ] Can I explain the dual-write problem and outbox?
- [ ] Can I state a business invariant before choosing a concurrency mechanism?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 22/46  
**Next:** Session 23
