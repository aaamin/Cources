# Session 30 — Design a Notification Service

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice queues, priority, retries, deduplication, provider isolation, preferences, and fan-out.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Requirements: push/email/SMS, user preferences, templates, priority, scheduling, delivery status.
- Producer API/event and idempotency.
- Queue/topic separation by priority/channel/tenant when useful.
- Worker pools and provider adapters.
- Retry/backoff/DLQ; provider-specific rate limits.
- Deduplication and at-least-once semantics.
- User preference lookup and suppression.
- Observability: queue age, send latency, provider errors, delivery result.
- Emergency/bulk campaigns versus transactional notifications.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design the happy path and failure path for a password-reset email.
- [ ] Change request: send 100M emergency notifications without delaying password resets.
- [ ] Show how a provider outage is isolated.

## **Must Remember for the Interview**

- **Transactional and bulk notifications need isolation so one campaign cannot starve critical traffic.**
- **At-least-once queues require deduplication/idempotent send logic where duplicate messages are unacceptable.**
- **Provider retries need backoff and provider-specific rate limits.**
- **Preferences and unsubscribe rules are correctness/compliance inputs, not decoration.**
- **Queue age is often a better user-impact signal than queue size alone.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Producer → durable queue → channel workers → provider adapter.**
- **Separate priority/channel workloads when isolation matters.**
- **Retry transient failures; DLQ poison/permanent failures; reconcile uncertain delivery.**
- **Deduplicate by notification/event ID.**
- **Protect password resets and other critical notifications from bulk fan-out.**

## Self-Test Before Marking This Session Complete

- [ ] Did I define delivery semantics?
- [ ] Did I isolate priority traffic?
- [ ] Did I handle provider rate limits/outages?
- [ ] Did I handle duplicates?
- [ ] Did I define observability and DLQ/replay?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Send an emergency campaign without delaying password-reset notifications.

**Score using the 40-point rubric.**


---

**Progress:** Session 30/46  
**Next:** Session 31
