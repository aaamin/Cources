# Session 38 — Design Ticketmaster

**Phase:** Phase 3 — Advanced System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice a correctness-first system under extreme contention, payment uncertainty, and oversell risk.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Search/browse path versus inventory/booking path.
- Seat inventory source of truth.
- Hold/reservation state machine and expiration.
- Concurrency control: atomic update/unique constraint/locking/versioning.
- Payment workflow and idempotency.
- Saga/reconciliation for payment/reservation partial failure.
- Waiting room/admission control under burst traffic.
- Audit trail.
- Cache reads but never treat stale cache as authoritative for seat ownership.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Write the invariant first: one seat cannot be sold to two orders.
- [ ] Change request: event sells out in 30 seconds while payment provider is slow.
- [ ] Walk every state transition through timeout, retry, and expiration.

## **Must Remember for the Interview**

- **Correctness beats raw availability for the seat-allocation invariant.**
- **A 'check availability then update' flow is racy unless the update is atomic/locked/versioned.**
- **Use short-lived holds to separate browsing intent from final purchase.**
- **Payment timeout creates an uncertain state; idempotency and reconciliation are required.**
- **Admission control/waiting room protects the critical inventory subsystem during extreme bursts.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Invariant: never sell the same seat twice.**
- **Model states: AVAILABLE → HELD → SOLD, with expiration/release paths.**
- **Use DB atomicity/constraints/locks before adding distributed locks.**
- **Payment and reservation form a distributed workflow; reconcile uncertain outcomes.**
- **Cache search/read data, but validate booking against authoritative inventory.**

## Self-Test Before Marking This Session Complete

- [ ] Did I state the invariant first?
- [ ] Did I make seat transition atomic?
- [ ] Did I define hold expiration?
- [ ] Did I handle payment timeout/retry?
- [ ] Did I protect the system during a ticket-sale spike?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Critical invariant:** Never sell the same seat twice.

**Required change request:** Global event sells out in seconds while payment is slow.

**Score using the 40-point rubric.**


---

**Progress:** Session 38/46  
**Next:** Session 39
