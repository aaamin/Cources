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


## Important interview ideas

> **Important:** Correctness questions become much easier when you first state the invariant. Examples: **never charge twice**, **never sell one seat twice**, **every confirmed order has a successful payment**.

### Idempotency record lifecycle

For a create/payment API:

```text
idempotency_key
request_hash
status: IN_PROGRESS / SUCCEEDED / FAILED
result_reference
expires_at
```

On retry, the service checks the key. If the same request already succeeded, return the stored result. If the key is reused with different input, reject it.

Keys need retention long enough to cover realistic retries.

### Optimistic vs pessimistic locking

Optimistic locking:

```text
UPDATE seat
SET state='HELD', version=version+1
WHERE seat_id=? AND version=? AND state='AVAILABLE'
```

Only one concurrent writer succeeds. It works well when conflicts are not constant.

Pessimistic locking obtains an exclusive lock first. It can simplify very high contention but creates wait/deadlock/failure concerns.

### Lease and fencing

A worker owns a lease until time T. If it pauses, the lease may expire and another worker takes over. The old worker can resume later and incorrectly write.

A fencing token solves this by giving each lease acquisition a higher number. The protected storage accepts only writes with the newest token.

### Saga design

A saga defines local transactions and compensation:

```text
reserve inventory
→ charge payment
→ create shipment
```

Failure after payment may trigger refund + inventory release. Compensation is not “rollback time”; external effects may require new business actions.

### Transactional outbox in detail

Inside one DB transaction:

```text
UPDATE order status
INSERT outbox_event
COMMIT
```

A publisher reads outbox rows and sends events. If it crashes after publish before marking sent, the event can publish twice, so consumers remain idempotent. The outbox prevents **lost event after committed DB change**, not duplicate delivery.

### Reconciliation closes the loop

Periodic jobs compare order/payment/inventory state and repair anomalies:

- payment captured but order pending;
- order confirmed but notification missing;
- inventory held after expired order.

This is how real distributed workflows survive rare partial failures.

## Worked scenario — duplicate payment retry

Client sends `POST /pay` with key `K123`. Provider charges successfully but the response is lost. Client retries.

Without idempotency → second charge.

With idempotency → Payment Service finds successful `K123` and returns the original payment ID. The order workflow then continues or reconciles if previous state update failed.

## Interview questions and model answers

### Q1. “Exactly once?”

I avoid promising exactly-once transport. A worker can perform an external effect then crash before acknowledgement. I design at-least-once processing with idempotent business operations, unique constraints, transaction boundaries, and reconciliation so the **business effect** is effectively once.

### Q2. “Saga vs distributed transaction?”

A saga is appropriate when services own independent databases and can tolerate intermediate states/compensation. A distributed transaction can give stronger atomicity but increases coordination and availability/operational complexity. Keep one DB transaction when possible.

### Q3. “When use optimistic locking?”

When contention is moderate and conflicts can be retried. It avoids long-held locks. For extremely contended scarce inventory, pessimistic/serialized techniques may be easier to reason about.

### Q4. “What does outbox solve?”

It atomically records the business state change and intent to publish an event in one local DB transaction, preventing the dual-write case where DB commits but event publication is permanently lost.

## Common mistakes to avoid

- No invariant stated.
- Idempotency key accepted with different request body.
- Distributed lock with no lease/fencing story.
- Saga compensation assumed infallible.
- Outbox assumed exactly-once.
- No reconciliation.
- “Check then write” race without conditional update/constraint.

## Short revision note

Correctness toolkit: **invariant → transaction/conditional write → idempotency → lease/fencing if needed → Saga/outbox across services → reconciliation**.

## Topics to revise

- [ ] race conditions
- [ ] optimistic/pessimistic locking
- [ ] idempotency key lifecycle
- [ ] leases/fencing
- [ ] Saga/compensation
- [ ] dual-write problem
- [ ] transactional outbox / CDC
- [ ] reconciliation

## Interview-ready synthesis

### A strong 60–90 second explanation

I start correctness-heavy designs by writing the invariant. I use database transactions/conditional writes for local concurrency, idempotency keys for retries, leases plus fencing for temporary ownership, and Saga/outbox/reconciliation when a workflow crosses service/database boundaries.

### How this topic connects to the wider system

- Correctness: conditional state transitions prevent races and duplicate effects.
- Reliability: reconciliation repairs partial/ambiguous outcomes after crashes.
- Architecture: transactional outbox connects local DB change to asynchronous events safely.
- Scalability: avoid global locks; keep invariant scope as local as possible.

### Revision flashcards with answers

**Optimistic locking?**  
Update succeeds only if version/state still matches; loser retries.

**Idempotency key?**  
Stable logical-operation ID used to return existing result on retry.

**Saga?**  
Sequence of local transactions with compensating actions for cross-service workflow.

**Outbox?**  
Business update and pending event are committed in one DB transaction.

**Fencing token?**  
Increasing ownership number that lets storage reject stale lease-holder writes.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Checkout: idempotency prevents duplicate orders/payments; Saga handles inventory/payment across services.
- Job scheduler: leases + fencing prevent stale worker from overwriting new owner state.
- Notification outbox: committed order event cannot be lost between DB write and publish.

### When not to overuse this idea

Do not reach for a distributed lock when a local transaction/conditional write on the authoritative row can enforce the invariant more simply.

### A good interviewer sentence

> “I would use this only because the current requirement/workload creates the specific problem it solves. If that assumption changes, I would simplify or choose the alternative.”

This sentence captures an important L4 behavior: architecture is conditional, not dogmatic.

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
