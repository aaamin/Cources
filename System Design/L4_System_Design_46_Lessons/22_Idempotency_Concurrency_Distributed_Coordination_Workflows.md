# Session 22 — Idempotency, Concurrency, Distributed Coordination & Workflows

## Outcome

You should be able to protect business invariants under concurrent requests, retries, duplicate events, worker failure, and multi-service workflows using atomic constraints, optimistic/pessimistic locking, compare-and-set, leases, fencing, idempotency keys, Saga, transactional outbox, CDC, and reconciliation.

## Start With the Invariant

Do not begin with locks.

State what must remain true.

Examples:

```text
Seat cannot be sold twice.
Payment for one order cannot be captured twice.
Coupon can be redeemed once per user.
Job result from a stale worker must not overwrite a newer run.
```

Then choose mechanisms.

## Race Conditions

Two operations interleave and produce wrong result.

Example:

```text
stock = 1

A reads 1
B reads 1
A writes 0
B writes 0
```

Both think purchase succeeded.

Need atomicity/concurrency control.

## Unique Constraints

If invariant is uniqueness, database can enforce it.

Example:
```text
UNIQUE(user_id, coupon_id)
```

Two concurrent redemptions:
- one insert succeeds;
- one gets conflict.

This is often simpler than a distributed lock.

## Optimistic Locking

Assume conflicts are uncommon.

Store version:

```text
Order(id, status, version)
```

Update:

```sql
UPDATE orders
SET status='PAID', version=version+1
WHERE id=? AND version=?;
```

If affected rows = 0:
- someone changed it;
- reread/retry or reject.

Good when:
- contention low/moderate;
- work can retry.

## Pessimistic Locking

Lock resource before modifying.

Example:
```sql
SELECT ... FOR UPDATE
```

Good when:
- conflict likely;
- critical section short.

Costs:
- blocking;
- deadlocks;
- lower concurrency.

Keep lock duration short.

## Compare-and-Set

Update only if current value/version matches expectation.

Core primitive behind many optimistic designs.

Example:
```text
AVAILABLE → HELD
only if state is still AVAILABLE
```

## Idempotency Keys

Client retries after timeout:

```text
POST /payments
Idempotency-Key: abc123
```

Server stores:
```text
key → request identity/result/status
```

If same key repeats:
- return existing result rather than perform again.

Important questions:
- scope (user/merchant/global)?
- expiry/retention?
- same key with different payload?
- in-progress concurrent duplicate?
- result storage?

## Idempotency-Key Race

Two identical requests arrive simultaneously.

Bad:
```text
check key absent
both execute payment
then both save key
```

Need atomic uniqueness/transaction around key registration.

Example:
```text
INSERT idempotency_key UNIQUE
```
Only one request becomes owner; other waits/reads result.

## Duplicate Events

Consumer should use:
- event ID;
- business key/version;
- unique constraint;
- idempotent state transition.

Avoid “remember every event forever” if unbounded; choose retention based on duplicate horizon or business record semantics.

## Distributed Locks

A service may acquire lock in shared system.

Useful sometimes for:
- leader-like singleton work;
- coarse coordination;
- preventing duplicate expensive rebuild.

But distributed locks are tricky.

Questions:
- lease expiration?
- process pauses?
- network partition?
- lock service unavailable?
- stale holder?

For DB-local invariant, DB atomic operation is often safer.

## Leases

Lock with expiry.

```text
Worker A owns job until 12:00:30
```

If A dies, another worker can take over after expiry.

Need renewal/heartbeat for long work.

Problem: A may pause, lease expires, B acquires, then A resumes.

Now two workers believe they can commit.

## Clock Skew

Different machines' clocks are not perfectly aligned.

Avoid correctness designs that assume:
```text
my local wall clock proves my lease is valid globally
```

Use coordination service/server time/version/epoch semantics where needed.

## Fencing Tokens

Each lease acquisition gets monotonically increasing token:

```text
A gets token 41
lease expires
B gets token 42
```

Storage/resource accepts writes only if token >= last seen.

A resumes with 41:
```text
rejected as stale
```

This protects against stale lock holders.

At L4, understand the idea; you do not need to implement consensus behind token generation.

## Leader Election Recognition

Sometimes one worker/node should coordinate:
- scheduler;
- partition controller;
- singleton cleanup.

Election system chooses one leader/epoch.

Again, stale leader must be fenced/prevented from committing after new leader exists.

## Dual-Write Problem

Service must update DB and publish event.

Naive:

```text
1. update DB
2. publish event
```

Crash between:
- DB committed;
- event never published.

Reverse order:
- event published;
- DB transaction fails.

Need a method to tie durable state and event publication.

## Transactional Outbox

Within one DB transaction:

```text
BEGIN
update order
insert OutboxEvent(OrderPaid)
COMMIT
```

Separate publisher reads outbox and publishes.

If publisher retries, duplicate publish is possible, so consumers remain idempotent.

Outbox converts “lost event” into retryable publication.

## CDC — Change Data Capture

Infrastructure reads DB change log/outbox changes and publishes to stream.

Useful for:
- search indexing;
- analytics;
- cache invalidation;
- integration.

Recognition-level understanding is enough.

## Saga Pattern

For workflow across services where one DB transaction cannot cover everything.

Example booking:

```text
Create order
 ↓
Reserve inventory
 ↓
Charge payment
 ↓
Confirm order
```

If payment fails:
```text
release inventory
cancel order
```

These compensating actions form a Saga.

Important:
- compensation is business logic, not perfect rollback;
- steps can fail;
- retries/idempotency required;
- workflow state must be durable.

## Orchestration vs Choreography

### Orchestration
Coordinator tells steps what to do.

Pros:
- workflow visible;
- centralized state.

Cons:
- coordinator complexity.

### Choreography
Services react to events.

Pros:
- loose coupling.

Cons:
- workflow becomes hard to see/debug;
- event cycles/implicit dependencies.

Either can be reasonable.

## Reconciliation

Distributed systems occasionally end in uncertain/inconsistent states.

Example:
- payment provider shows captured;
- order DB says pending.

Reconciliation job:
- compare source/provider states;
- repair/escalate discrepancies.

This is not a hack; it is an important reliability mechanism for money/workflows.

## CQRS/Event Sourcing Recognition

CQRS:
- separate command/write model from read model.

Event sourcing:
- state reconstructed from event history.

Know recognition/trade-offs only. Do not introduce these in normal designs without strong reason.

## Worked Example — Payment

Requirements:
- one order charged once;
- client may retry;
- provider may timeout;
- webhook may duplicate.

Design:
1. `PaymentRequest(order_id UNIQUE, idempotency_key UNIQUE, state)`.
2. Client POST uses idempotency key.
3. Atomically claim key/request.
4. Call provider with provider idempotency key.
5. Persist provider transaction/result.
6. Outbox `PaymentCompleted`.
7. Webhook handler idempotent on provider transaction/event.
8. Reconciliation checks pending/unknown payments.

If HTTP timeout occurs after provider captured:
- do not blindly charge again;
- query/reconcile using same provider idempotency/business reference.

## Small Design Drills

1. Why is a DB unique constraint often better than a distributed lock for “coupon once per user”?
2. What happens if a lease expires while old worker is paused?
3. What does fencing token protect?
4. Why doesn't transactional outbox guarantee consumers see event once?
5. What is compensation in a Saga?
6. Client payment request times out. Why is blindly retrying with a new key dangerous?
7. What is reconciliation for?

<details>
<summary>Answer key</summary>

1. The DB directly enforces the invariant atomically with less distributed coordination.
2. New worker may take over; old worker can resume and become stale concurrent writer.
3. Rejects writes from stale lease/leader holders after a newer token exists.
4. Publisher may publish duplicate after crash/retry; consumers still need idempotency.
5. Business action that semantically undoes/offsets a prior completed step, e.g. release inventory/refund.
6. Provider may already have charged; new key can create second payment.
7. Detect/repair discrepancies after uncertain partial failures.

</details>

## Common Interview Mistakes

- Using distributed lock for every race.
- “Exactly once payment” without idempotency/reconciliation.
- Check-then-insert race on idempotency key.
- Lock with no lease expiry.
- Lease with no stale-holder protection.
- Holding DB transaction across remote workflow.
- Dual write DB + broker with no outbox/CDC strategy.
- Saga described as automatic rollback.
- No durable workflow state.
- No reconciliation for external money systems.

## Must Remember

- **Start from the invariant.**
- **Use DB constraints/atomic operations when they can enforce it locally.**
- **Optimistic locking uses versions/CAS; pessimistic locking blocks.**
- **Idempotency keys must themselves be claimed atomically.**
- **Leases allow recovery but create stale-worker risk.**
- **Fencing tokens reject stale holders.**
- **Transactional outbox prevents DB-update/event-publication gaps.**
- **Outbox still permits duplicate publish.**
- **Saga uses compensating business actions across services.**
- **Reconciliation handles uncertain distributed reality.**

## Interview Revision Summary

Correctness checklist:

```text
Invariant?
Concurrent requests?
Atomic DB constraint?
Optimistic/pessimistic?
Retry?
Idempotency key?
Duplicate event?
Lease?
Stale worker?
Fencing token?
DB + event dual write?
Outbox/CDC?
External side effect?
Saga?
Reconciliation?
```

## Explain Without Notes

Design the payment portion of an e-commerce checkout where clients retry, provider responses can time out, webhooks duplicate, and payment must not be captured twice.

## Completion Checklist

- [ ] I start from invariants.
- [ ] I can use unique constraints/CAS/locks appropriately.
- [ ] I design idempotency-key atomicity.
- [ ] I understand leases/clock skew/fencing.
- [ ] I understand outbox/CDC.
- [ ] I understand Saga/compensation.
- [ ] I use reconciliation for uncertain external state.
