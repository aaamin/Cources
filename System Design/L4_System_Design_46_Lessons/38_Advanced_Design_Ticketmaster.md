# Session 38 — Advanced Design — Ticketmaster

## Interview Prompt

> Design a ticket-booking system for assigned seats. Users browse availability, hold seats temporarily, pay, and receive confirmed tickets.

### Critical Invariant

> **Never sell the same seat twice.**

### Mid-Interview Change

> A globally popular event sells out in seconds while the payment provider becomes slow.

Treat this as a correctness-heavy interview.

---

# STOP — Complete Your Design First

You must define:
- source of truth for inventory;
- seat states;
- hold expiry;
- concurrent hold behavior;
- payment timeout/unknown result;
- idempotency;
- waiting room/admission;
- reconciliation/audit.

---

# Interviewer Pressure Pack

### Pressure 1
Two users click the same seat within 5 ms.

### Pressure 2
Seat hold succeeds, payment provider charges successfully, but your request times out before you receive the provider response.

### Pressure 3
Hold expires while payment callback arrives late.

### Pressure 4
A million users refresh the same event seating map.

### Pressure 5
The primary region fails during the onsale.

Respond before reading the reference reasoning.

---

# Reference Reasoning

## 1. Requirements

Core:
- browse event/seat map;
- view availability;
- hold specific seats for short time;
- pay;
- confirm;
- release expired hold;
- ticket retrieval.

Non-functional:
- zero oversell;
- high read scale;
- huge synchronized burst;
- durable purchase/audit;
- payment uncertainty handled;
- graceful admission during overload.

The invariant determines architecture more than QPS.

## 2. Seat State

Authoritative state:

```text
AVAILABLE
HELD
SOLD
```

Fields:

```text
SeatInventory
- event_id
- seat_id
- state
- hold_id
- hold_expires_at
- order_id
- version
```

Unique primary key:
```text
(event_id, seat_id)
```

## 3. Seat Map Read Path

Seat-map availability can be cached/read-replicated approximately.

Important:
> Cached “available” is a hint, not the authority for final hold.

User can click a seat shown available and still receive:
```text
seat no longer available
```

This is acceptable and avoids making millions of map reads strongly coordinated.

## 4. Atomic Hold

Do not:

```text
SELECT state
if AVAILABLE
UPDATE HELD
```

without concurrency protection.

Use conditional update/transaction:

```sql
UPDATE seat_inventory
SET state='HELD',
    hold_id=:hold,
    hold_expires_at=:expiry,
    version=version+1
WHERE event_id=:event
  AND seat_id=:seat
  AND (
       state='AVAILABLE'
       OR (state='HELD' AND hold_expires_at < current_time)
  );
```

One transaction wins.

For multiple seats:
- sort seat IDs to avoid lock-order deadlocks;
- hold all in one transaction when same DB partition;
- if any unavailable, rollback all (depending product).

Partition event inventory so a single booking stays local if possible.

## 5. Hold

Hold creates temporary ownership:

```text
Hold
- hold_id
- user_id
- event_id
- expires_at
- state
```

Timer does not need to execute exactly at expiry.

Correctness check:
```text
hold is valid only if now < expires_at
```

Background sweeper releases physically later.

## 6. Clock / Expiry

Use DB/server-authoritative time for seat state transition rather than trusting client clock.

Hold extension policy must be explicit.

Client showing “12 seconds left” is UI; server is authority.

## 7. Order and Payment State

Separate order/payment state:

```text
Order
- id
- hold_id
- state: PENDING_PAYMENT / CONFIRMED / CANCELLED / REVIEW
- amount

Payment
- order_id
- provider_reference
- state
- idempotency_key
```

Do not convert seat to SOLD merely because client says payment succeeded.

## 8. Payment Flow

1. validate hold is still owned/valid;
2. create payment attempt with idempotency key;
3. call provider;
4. provider returns success/decline/timeout;
5. success → transactionally mark order confirmed + seats sold;
6. emit outbox events.

But timeout is hard.

## 9. Payment Timeout / Unknown Outcome

Scenario:
- provider captured;
- network response lost.

State:
```text
PAYMENT_UNKNOWN / PENDING_CONFIRMATION
```

Do **not** retry with a new payment identity.

Use same provider idempotency/business reference:
- query provider;
- wait for signed webhook;
- reconciliation.

Hold may need temporary extension while payment outcome is unknown.

## 10. Hold Expiry vs Late Payment

This needs explicit policy.

Safer workflow:
- once payment attempt begins before valid hold expires, transition seat/order to `PAYMENT_PENDING` reservation that cannot be stolen immediately;
- give payment resolution grace window;
- if payment ultimately fails → release;
- if payment succeeds → sell.

Do not let seat be sold to user B while user A's payment may already have been captured.

Alternative:
- authorize payment first then final inventory transaction, but inventory can disappear between steps unless provider auth/void flow supports it.

State machine is clearer than timing assumptions.

## 11. Idempotency

Create order:
```text
client_request_id UNIQUE
```

Payment:
```text
order_id/provider payment key unique
```

Provider webhook:
```text
provider_event_id UNIQUE
```

Order confirmation:
- idempotent transition;
- duplicate success webhook does not issue second ticket.

## 12. Waiting Room

At onsale, allowing 5M users to hit inventory DB is unnecessary.

Front-door:

```text
Users
  ↓
Waiting Room / Admission
  ↓ bounded admitted sessions
Booking APIs
```

Goals:
- protect inventory/payment;
- fairness;
- bot control;
- stable latency.

Waiting room may use signed queue/admission token.

Do not promise mathematically perfect fairness without requirement.

## 13. Rate Limit / Anti-Bot

Per:
- account;
- IP/device;
- event;
- session.

Controls:
- CAPTCHA/challenge;
- purchase limits;
- suspicious automation;
- signed admission token.

Ticket systems are abuse-heavy.

## 14. Cache Strategy

Cache:
- event details;
- seating layout;
- approximate availability snapshots.

Do not cache authoritative hold decision.

Invalidation:
- stream seat-state updates to cache/clients;
- users tolerate occasional stale map because hold is authoritative.

At high scale, aggregate availability:
```text
section has 50 seats
```
can be more cacheable than every seat update.

## 15. Event Partitioning

Each popular event can be very hot.

Possible:
- partition by event + section;
- transaction for multiple seats may cross section if user chooses arbitrary seats;
- route seats to owning shard;
- isolate blockbuster event to dedicated capacity.

Do not shard every event across hundreds of partitions if transactional booking of adjacent seats becomes impossible.

Choose partition granularity based on booking access pattern.

## 16. Oversell Protection Pattern

Weak read:
```text
seat map says available
```

Strong commit:
```text
conditional hold
```

This is similar to ride matching:
**approximate discovery, strong narrow commit.**

## 17. Payment Provider Slow

Use:
- timeout;
- idempotency;
- bounded concurrency;
- circuit breaker where safe;
- payment-pending queue;
- do not retry blindly;
- waiting room reduces new admissions;
- extend already-started payment reservations.

If provider cannot process current throughput, accepting unlimited new holds just creates mass expiry/poor UX.

Admission should adapt to payment capacity.

## 18. Regional Strategy

For one event, choose a write home region/authority for inventory.

Global users can:
- read event data from edges;
- booking writes route to event's authoritative region.

Why not active-active inventory writes?
- concurrent seat ownership across regions is difficult;
- global strong coordination adds similar latency anyway.

If primary region fails:
- pause new bookings;
- promote synchronized secondary;
- use fencing/epoch;
- ensure old primary cannot resume writes as authority.

Correctness > availability for seat ownership.

## 19. Audit / Reconciliation

Keep immutable-ish audit trail:
- hold creation/expiry;
- order transitions;
- payment references;
- ticket issuance/refund.

Reconciliation:
- provider captured but order not confirmed;
- sold seat with missing ticket;
- expired hold still reserved;
- duplicate webhook.

Money systems need repair loops.

## 20. Observability

Per event:
- waiting-room depth/age;
- admitted sessions/sec;
- hold success conflict rate;
- DB lock/transaction latency;
- active holds;
- expiry rate;
- payment p95/error/unknown rate;
- order confirmation latency;
- reconciliation mismatches.

## Common Mistakes

- Cache decides seat sale.
- Read-then-write race.
- Lock all seats globally.
- Payment HTTP timeout treated as failure.
- New payment key on retry.
- Hold expires and seat resells while prior charge uncertain.
- Waiting room mentioned but not connected to backend capacity.
- Active-active regional seat writes with no coordination.
- No audit/reconciliation.
- Exact expiry timer required for correctness.

## Must Remember

- **Seat ownership invariant dominates design.**
- **Availability cache is a hint; conditional hold is authority.**
- **Hold expiry should be validated by authoritative time/state.**
- **Payment timeout means unknown, not failed.**
- **Use one idempotent payment identity.**
- **Introduce PAYMENT_PENDING/grace to avoid late-payment oversell.**
- **Waiting room/admission protects scarce inventory/payment capacity.**
- **One event should have one authoritative write model unless global consensus is intentionally used.**
- **Reconciliation is mandatory for payment uncertainty.**

## Repair Exercise

From a blank page, solve only:

> User A begins payment with 5 seconds left on hold. Provider captures at T+4s. Your server gets no response. At T+6s user B tries to hold the seat. At T+8s provider webhook says A paid.

Write the state transitions that prevent oversell and double charging.
