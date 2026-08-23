# Lesson 22 — Idempotency, Transactions, Concurrency & Distributed Workflows

**Phase:** Fundamentals  
**Session:** 22/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn how to preserve business invariants under retries, concurrent operations, crashes, and workflows spanning multiple services.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Race conditions

A race occurs when correctness depends on timing. Two buyers can both read `seat=available` and both attempt to reserve it. The system must make the state transition atomic or detect conflict.

## Optimistic vs pessimistic concurrency

Optimistic concurrency uses a version/CAS and rejects an update if another writer changed the row. It works well when conflicts are uncommon. Pessimistic locking reserves exclusive access first, simplifying high-contention correctness but increasing waiting and lock-failure complexity.

## Idempotency

An idempotent operation can repeat without creating an extra business effect. Payment APIs commonly accept an idempotency key so a client that times out can retry and receive the original result rather than being charged twice.

## Leases, locks, fencing

Distributed locks need expiry so abandoned ownership can recover. But an old worker may continue after its lease expires. Fencing tokens give newer owners larger versions so downstream storage can reject stale-owner writes.

## Saga and compensation

A multi-service workflow cannot always use one ACID transaction. A saga performs local transactions and compensates earlier steps if later steps fail: reserve inventory → charge payment → create shipment; if shipment fails, refund and release inventory.

## Transactional outbox and dual writes

Updating a DB and publishing an event are two writes. If the DB commit succeeds but publishing fails, downstream state diverges. The outbox stores business change + event record in one local transaction; a publisher later reliably sends the event. CDC can achieve similar propagation.

## Reconciliation

Compensation and event delivery can also fail. Periodic reconciliation compares expected state and repairs inconsistencies. Mature distributed systems assume uncertain states will occasionally need repair.

## Worked example — checkout with payment retry

Use a client idempotency key. Reserve inventory with a conditional/transactional update. Charge payment idempotently. Persist order state. Publish `OrderConfirmed` through an outbox. If the service crashes after payment succeeds, recovery queries payment status and reconciles instead of charging again.

## Interview lens

For correctness-heavy designs, state the invariant first—“never charge twice,” “never oversell”—then explain how concurrency, retry, and partial failure preserve it.

## What to remember

Distributed correctness comes from explicit state transitions, idempotency, conflict control, compensation, durable event propagation, and reconciliation.

## Review after reading

1. What is a race?
2. Optimistic locking?
3. Why idempotency keys?
4. What dual-write problem does outbox fix?
5. Why reconciliation?

## Deeper study notes

### Idempotency record lifecycle

An idempotency table can map `(caller, key)` to request fingerprint, state, and final response. If the same key arrives with different payload, reject it; otherwise return the recorded result. Keep entries long enough to cover realistic client retry windows and business settlement periods.

### Optimistic locking example

A row contains `version=7`. Both buyers read it. Buyer A updates with `WHERE version=7`, setting version 8; one row changes. Buyer B's identical condition now changes zero rows, so B knows it lost and retries or reports conflict. This avoids holding a lock while the user thinks.

### Sagas need explicit states

Do not model a saga as “call A then B then C.” Store durable workflow state so recovery knows which steps completed. Each step and compensation must be idempotent. A crash should resume from persisted state rather than restart blindly.

### Outbox delivery is at-least-once

The outbox publisher can publish an event and crash before marking it sent, so the event may publish twice. Consumers therefore still need idempotency/deduplication. The outbox solves **loss between DB commit and publish**, not all duplicate problems.

### Common mistakes

- Using a distributed lock when a database conditional write would be simpler.
- Assuming a timeout means payment failed.
- Making compensation non-idempotent.
- Deleting workflow state too early for reconciliation.
- Claiming “exactly once” without defining the business effect and storage boundary.


## Personal notes

```text
Concepts that are clear:

Concepts to revisit:

Three things to remember:
1.
2.
3.

Questions for later:
```

---

**Next:** Lesson 23 — API & Event Contract Design
