# Lesson 30 — Design a Notification Service

**Phase:** Guided Design  
**Session:** 30/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Send push, email, and/or SMS.
- Respect user preferences and templates.
- Retry transient provider failures.
- Prevent bulk campaigns from starving critical messages.

## 2. Scale and workload shape

Estimate notifications/day, peak fan-out, provider quotas, and acceptable delay per class. Traffic is often bursty: a marketing campaign can create millions of messages at once, while password resets require low latency.

## 3. API / contract surface

```http
POST /v1/notifications
```

Or internal event:

```text
notification_id
user_id
template_id
channels
priority
dedupe_key
schedule_at
```


## 4. Data model

```text
Preference(user_id, channel, enabled)
Notification(id, user_id, type, status, created_at)
DeliveryAttempt(notification_id, channel, provider, status, attempt)
```

Templates and provider configuration are separate metadata.

## 5. High-level architecture

```text
Producer
  ↓
Notification API/Event
  ↓
Priority Queues
  ↓
Channel Workers
  ├─ Push Provider
  ├─ Email Provider
  └─ SMS Provider
```

Workers fetch current preferences, render templates, enforce quotas, call providers, and record status.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Priority isolation

Separate queues/worker capacity for transactional and bulk messages. This is a bulkhead: a 100M-user campaign cannot consume all password-reset capacity.

### Retry and deduplication

A provider can accept a message and then timeout. Retrying may deliver twice. Use stable notification IDs and provider idempotency when available.

### Provider abstraction

Adapters isolate provider-specific APIs. Per-provider circuit breakers and fallback routing keep one provider outage from blocking all channels.

## 7. Failure modes and recovery

- Provider outage: queue depth/age grows; alert and possibly fail over.
- Poison payload: retry limit → DLQ.
- Duplicate upstream event: dedupe by notification ID.
- Preference changed after enqueue: often re-check at send time.
- Bulk burst: admission/priorities preserve critical traffic.
- Provider rate limit: per-provider token bucket and scheduled retry.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Exactly-once human delivery is rarely achievable. Aim for at-least-once processing, stable IDs, dedupe, observable delivery state, and priority isolation.

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

### Why notification is naturally asynchronous

The initiating product action should not wait for a third-party SMS or email provider. The business service records the event and hands off durable notification intent. Providers can then be slow, unavailable, or rate-limited without making checkout/password-reset creation fail.

### Fan-out strategy

A notification request may expand into channels:

```text
Notification intent
   ├─ email delivery
   ├─ push delivery
   └─ SMS delivery
```

Expansion can happen before channel queues or inside an orchestration worker. Keep a stable notification ID so all channel attempts can be correlated and deduplicated.

### Scheduling and rate limits

Scheduled notifications need a delay/scheduler mechanism. Do not keep millions of sleeping application timers. Persist `send_at` and move due items into ready queues. Provider-specific token buckets/quotas prevent one provider from returning mass 429 errors.

### Delivery status semantics

“Sent to provider” is not always “delivered to user.” Push/email/SMS providers expose different acknowledgements. Define states carefully, for example:

```text
QUEUED → SUBMITTED → PROVIDER_ACCEPTED → DELIVERED? / FAILED
```

Only claim guarantees you can observe.

### Common interview mistakes

- One shared FIFO queue for emergency and marketing messages.
- Infinite retries that repeatedly spam users.
- Checking user preference only when queued even if delivery occurs hours later.
- Treating provider success as guaranteed human receipt.
- Failing the business transaction because notification delivery failed.

### Reusable patterns learned

Priority queues, bulkheads, provider adapters, at-least-once processing, dedupe, scheduling, external-rate-limit handling, and observability by state transition.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 31 — Design WhatsApp / Messenger
