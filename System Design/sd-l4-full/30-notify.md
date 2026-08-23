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


## Detailed reference design

### Clarify notification semantics

Ask:

- channels: push/email/SMS/in-app?
- immediate vs scheduled?
- transactional vs marketing?
- user preferences?
- priority/fairness?
- delivery status required?

The central challenge is not merely fan-out. It is **reliable asynchronous delivery through external providers while keeping critical traffic isolated**.

### Architecture with priority isolation

```text
Producers
   ↓
Notification API / Event Ingest
   ↓
Preference + Template resolution
   ↓
Priority Router
  ├─ Critical Queue  → critical workers
  ├─ Normal Queue    → normal workers
  └─ Bulk Queue      → campaign workers
                         ↓
                    Provider adapters
                   /      |       \
                Push    Email     SMS
```

Separate queue/worker budgets prevent 100M marketing jobs from starving OTP/password-reset delivery.

### Notification identity

Assign stable `notification_id` and often a business `dedupe_key`:

```text
password_reset:{user}:{reset_request_id}
```

Every channel attempt references the stable notification ID. This simplifies audit, dedupe, retry, and status.

### Preference timing

User preferences can change after enqueue. Usually check preferences near **send time**, not only event creation, especially for marketing. Critical security messages may override certain preferences according to product policy.

### Template versioning

Do not store only “template name.” A notification created today but delivered later should have deterministic content. Store template version or rendered immutable payload depending on policy. Localization and personalization may happen before enqueue or at send time.

### Provider adapters

Define a common internal interface:

```text
send(channel_message, idempotency_key) → provider_message_id/status
```

Each adapter handles provider auth, rate limits, error mapping, and idempotency support. This makes fallback/migration easier.

### Retry policy

Classify errors:

- 429/provider throttling → retry after delay;
- timeout/5xx → exponential backoff;
- invalid destination → permanent failure;
- bad template → DLQ/operator alert;
- provider auth failure → circuit breaker/config alert.

### Delivery semantics

You usually cannot guarantee exactly one email/SMS to a human. A provider can accept the message and your response can time out. Aim for:

- at-least-once processing;
- provider idempotency when possible;
- stable message IDs;
- bounded retries;
- duplicate-tolerant product semantics.

### Scheduling

Scheduled notifications can live in a durable schedule table/index by `send_at`. A scheduler moves due items into queues. Do not keep millions of in-memory timers.

## Failure walkthrough

### Email provider outage

Circuit breaker opens. Email queue grows. Push/SMS queues remain unaffected. Alert on oldest-message age. Critical email may fail over to secondary provider if configured.

### Emergency campaign

Bulk queue is rate-limited to provider quota and isolated. Critical queue has reserved worker/provider capacity. Campaign may take hours; password reset remains seconds.

### Duplicate source event

Dedupe by stable business key if duplicate notification is undesirable. Consumer processing remains idempotent.

### Preferences store slow

Cache preferences with bounded TTL or degrade by channel policy; do not let a non-critical preference lookup take down the entire notification system.

## Interviewer follow-ups

### “How do you know whether notification was delivered?”

Provider acceptance is not always human delivery. Track states such as `QUEUED → SENT_TO_PROVIDER → DELIVERED/BOUNCED` when providers supply callbacks. Be precise about what each status means.

### “How do you send to 100M users?”

Do not enqueue 100M in one synchronous request. Store campaign definition, enumerate audience in batches, create tasks progressively, apply rate limits, and isolate from transactional traffic.

### “What if user has three devices?”

Push channel expands one logical notification into device tokens. Invalid tokens are removed after provider feedback. Logical notification ID remains stable.

## Common interview mistakes

- One queue for every priority.
- Unlimited retries.
- “Exactly-once email.”
- No provider rate-limit handling.
- No user-preference strategy.
- Synchronous provider call from business API.
- No stable notification/dedupe ID.

## Short revision note

**Notification pattern:** durable event → preference/template → priority queues → channel workers → provider adapters → retry/DLQ/status. Isolation and idempotency are the key interview signals.

## Topics to revise

- [ ] priority/fairness
- [ ] queues/workers
- [ ] preferences
- [ ] template versioning
- [ ] provider adapter
- [ ] retry classification
- [ ] dedupe/idempotency
- [ ] provider callback/status
- [ ] campaign batching

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll separate notification creation from delivery. A durable notification/event enters priority-isolated queues, workers resolve current preferences/templates, and provider adapters handle retries/rate limits. The key guarantees are no lost critical work, controlled duplicates, and bulk traffic isolation.

## How the design evolves at 10×

At 10× campaigns, increase batch fan-out and workers but respect provider quotas. Add per-provider/priority partitioning, audience expansion workers, and secondary providers only where business value justifies it.

## Quick revision flashcards

**Critical vs bulk?**  
Separate queues/resource budgets so campaigns cannot starve OTP/reset.

**Exactly once?**  
Usually impossible at human/provider boundary; use idempotency/dedupe and bounded retry.

**Preference timing?**  
Often check near send time so recent opt-out is respected.

**Primary metric?**  
Oldest-message age per priority/channel plus provider success/rate-limit errors.

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

**Next:** Lesson 31 — Design WhatsApp / Messenger
