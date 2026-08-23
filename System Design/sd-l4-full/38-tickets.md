# Lesson 38 — Design Ticketmaster

**Phase:** Advanced Design  
**Session:** 38/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Search events and seat inventory.
- Place a temporary hold.
- Confirm after payment.
- Never sell the same seat twice.
- Survive flash-sale contention and provider slowness.

## 2. Scale and workload shape

Design for the **flash-sale peak**, not the daily average. Estimate concurrent buyers for one event, seats/event, hold duration, checkout rate, and payment latency. A waiting room/admission layer may be necessary even if global service traffic is manageable.

## 3. API / contract surface

```http
POST /v1/events/{id}/holds
POST /v1/holds/{id}/confirm
DELETE /v1/holds/{id}
GET  /v1/events/{id}/seats
```

Create/confirm calls should accept idempotency identifiers.

## 4. Data model

```text
Seat(event_id, seat_id, state, version)
Hold(hold_id, event_id, seat_id, user_id, expires_at, state)
Order(order_id, hold_id, payment_id, state)
```

Critical invariant: **one seat can have at most one active confirmed sale**.

## 5. High-level architecture

```text
Client → Waiting Room / Admission
            ↓
       Booking Service
        /           \
       v             v
Inventory DB     Payment Provider
       ↓              ↓
Hold Expiry / Reconciliation
```

Search/browse can be cached, but the inventory database/conditional state transition is authoritative for holds.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Contention

Use row lock, optimistic version/CAS, or conditional write so only one hold transition from AVAILABLE succeeds. “Check availability then write” without atomicity races.

### Hold expiration

Treat a hold as a lease with `expires_at`. Cleanup workers release expired state, but reservation logic should also check expiry so correctness does not depend on a perfectly punctual worker.

### Payment and saga

Payment may succeed while order confirmation persistence fails. Payment calls need idempotency; uncertain orders are reconciled by querying provider state and finishing/refunding as policy requires.

### Waiting room

Admission control limits active buyers so the inventory database sees sustainable contention instead of millions of concurrent transactions.

## 7. Failure modes and recovery

- Client retries hold: idempotency returns prior result.
- Two buyers race: atomic conditional update lets only one win.
- Payment timeout: query status; do not blindly charge again.
- Service crash after payment: reconciliation finishes order or refunds.
- Hold worker down: expired timestamp still invalidates hold.
- Flash sale overload: queue/waiting room sheds excess concurrency before DB.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Use strong correctness only around inventory/order transitions. Event browsing and approximate availability can be cached/eventually consistent. Narrowing the strong-consistency region preserves scale.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Browse availability is not booking truth

Users can browse cached seat maps or approximate availability. The final hold operation must consult authoritative inventory. This separates a highly scalable read path from a small correctness-critical write path.

### Atomic hold transition

Conceptually:

```sql
UPDATE seat
SET state='HELD', hold_id=?, expires_at=?
WHERE event_id=? AND seat_id=? AND state='AVAILABLE';
```

If affected rows = 1, the hold won. If 0, another buyer already changed the state. The exact syntax varies, but the key is an atomic conditional transition.

### Expiration without races

A seat can be `HELD` with an expiry timestamp. A new hold may claim it if the old hold is expired, using one atomic transaction/condition. A cleanup worker is then an optimization that removes expired holds proactively, not the only correctness mechanism.

### Payment uncertainty

Never model timeout as failure. Payment provider may have accepted the charge. Use stable payment idempotency key and query/reconcile status. Keep the order/hold in a durable intermediate state until certainty is reached.

### Event-level partitioning

Inventory for one popular event is a natural hotspot. Partitioning across events spreads normal load, but the single hottest event still needs concurrency control and admission. Further split by seat section while preserving each seat's unique owner if required.

### Common interview mistakes

- Cache as source of truth for seat availability.
- Check-then-write race.
- Releasing a hold solely because cleanup did not run yet.
- Retrying payment blindly after timeout.
- Scaling app servers while inventory DB is saturated by lock contention.
- Forgetting waiting room/load shedding for flash-sale arrival rate.

### Reusable patterns learned

Strong invariant isolation, conditional writes, leases/holds, idempotent payment, saga/reconciliation, waiting room admission, and read-vs-write consistency separation.


## Detailed reference design

### The invariant drives the architecture

> **Invariant:** one seat/inventory unit can be confirmed for at most one buyer for the same event/time.

Everything else—cache, search, queue, waiting room—is secondary to preserving this under high contention.

### Separate browse from booking

Browse/search can be cached and slightly stale:

```text
Event catalog / seat map cache
```

Final hold/confirmation must use authoritative inventory state. Showing “available” is not the same as guaranteeing availability.

### Seat state model

```text
AVAILABLE
  ↓ create hold
HELD(hold_id, user, expires_at)
  ├→ CONFIRMED
  └→ AVAILABLE after expiry/cancel
```

A transaction/conditional update ensures only one transition from AVAILABLE succeeds.

Example SQL-style idea:

```sql
UPDATE seats
SET state='HELD', hold_id=?, expires_at=?
WHERE event_id=? AND seat_id=? AND state='AVAILABLE';
```

Check affected row count. This avoids “read available then write” race.

### Hold expiration

Do not depend only on a background timer. Every transaction reading a hold checks `expires_at`. An expiration worker cleans/releases state for efficiency. If worker is delayed, correctness still holds.

### Payment workflow

A robust flow:

1. hold seat;
2. create order `PAYMENT_PENDING`;
3. call payment with stable idempotency key;
4. if payment succeeds, confirm seat/order transactionally or through recoverable workflow;
5. if payment fails, release hold;
6. reconciliation repairs uncertain cases.

Payment provider timeout creates unknown outcome; query/retry by same key.

### Flash-sale admission control

If 5M users hit 50k seats simultaneously, allowing all requests to hammer inventory DB is harmful.

Use waiting room/token admission:

```text
millions users
   ↓
virtual queue/admission
   ↓ controlled rate
booking service
```

The queue does not guarantee a seat; it controls load/fairness.

### Partitioning

Partition inventory by `event_id` or event/section so all contention for one event is localized. One mega event remains hot, but that is an inherent contention domain. Split by section/seat range if necessary while keeping each seat owned by exactly one partition.

### Cache usage

Cache event metadata, price displays, and approximate availability. Never use cache as authoritative “last seat” state. Invalidation can lag; final booking checks source.

## Failure walkthrough

### Client times out after hold creation

Retry with idempotency key returns existing hold. Otherwise user could create many holds unintentionally.

### Payment succeeds, service crashes

Reconciliation finds successful payment with pending order and either confirms reservation if hold still valid/reserved for that order or executes product-defined refund/repair. Never blindly charge again.

### Hold expiry worker is down

Transactions treat expired holds as available/reclaimable using `expires_at`; cleanup backlog is operational, not correctness-critical.

### Payment provider is slow

Hold may expire while payment is uncertain. Define policy: extend hold during active payment, or if payment confirms too late, refund. State machine must make this deterministic.

## Interviewer follow-ups

### “Should we use a distributed lock?”

A DB conditional update/transaction on the authoritative seat row is often simpler. A distributed lock adds lease/fencing failure modes and still needs durable state. Use it only if the storage primitive cannot enforce the invariant efficiently.

### “Can we sell general admission?”

Model an atomic remaining counter rather than individual seat. Conditional decrement if `remaining > 0`. High contention may use partitioned inventory allocations but final total must not oversell.

### “What about two regions?”

Avoid active-active writes to the same event inventory unless you have strong cross-region coordination. Route an event's booking authority to one home region while other regions serve browse traffic.

## Common interview mistakes

- Cache as source of truth for availability.
- Check-then-insert race.
- Hold cleanup timer required for correctness.
- Payment timeout treated as failed charge.
- No idempotency.
- Waiting room presented as inventory correctness mechanism.
- Active-active regional inventory with no conflict strategy.

## Short revision note

**Ticketing pattern:** cached browse → authoritative conditional hold → expiring lease → idempotent payment workflow → confirm/reconcile → waiting room protects hot inventory.

## Topics to revise

- [ ] seat invariant
- [ ] conditional write/lock
- [ ] hold state/expiry
- [ ] idempotent payment
- [ ] Saga/reconciliation
- [ ] waiting room
- [ ] event partitioning
- [ ] cache vs source of truth

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll start with the no-oversell invariant. Browse/seat maps can be cached, but the authoritative inventory store uses atomic conditional holds with expiry. Payment is idempotent and the order/hold workflow is reconciled after ambiguous failures. A waiting room protects hot inventory but does not provide correctness by itself.

## How the design evolves at 10×

At 10× flash-sale demand, admission control and event/section partitioning protect the critical DB. The real limit is contention on scarce inventory, so scale cannot remove the need for serialized/conditional ownership of each seat.

## Quick revision flashcards

**Hold correctness?**  
Conditional AVAILABLE→HELD transition plus expiry check.

**Payment timeout?**  
Unknown outcome; query/retry with same idempotency key.

**Waiting room purpose?**  
Load/fairness control, not seat ownership.

**Multi-region?**  
Prefer one authority/home region per event unless strong global coordination is justified.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 39 — Design a Metrics / Logging Platform
