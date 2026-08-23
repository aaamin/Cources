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
        /           Inventory DB     Payment Provider
      ↓
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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 39 — Design a Metrics / Logging Platform
