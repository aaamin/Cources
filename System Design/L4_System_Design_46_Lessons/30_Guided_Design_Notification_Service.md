# Session 30 — Guided Design — Notification Service

## Interview Prompt

> Design a notification platform that sends email, SMS, and push notifications for many product teams.

Change request:
> A huge emergency campaign must be sent without delaying password-reset notifications.

Attempt the full design before reading.

---

# STOP — Design First

Consider:
- API/events;
- preferences;
- queues;
- provider adapters;
- retries;
- deduplication;
- priority;
- quotas;
- provider failure;
- observability.

---

# Reference Reasoning

## 1. Requirements

Functional:
- accept notification request/event;
- channels: email/SMS/push;
- templates;
- recipient preferences;
- scheduled/immediate;
- delivery status;
- retries;
- deduplication.

Non-functional:
- durable acceptance for important notifications;
- provider isolation;
- high burst throughput;
- password reset low latency;
- marketing can be delayed;
- at-least-once acceptable with idempotency.

Clarify:
- guarantee: accepted vs delivered?
- ordering?
- campaign size?
- per-recipient/channel rate limits?
- tenant isolation?

## 2. API / Event

Synchronous API:

```http
POST /notifications
Idempotency-Key: ...
{
  "userId": "...",
  "type": "PASSWORD_RESET",
  "channels": ["email"],
  "templateId": "...",
  "data": {...}
}
```

Or upstream services emit:
```text
PasswordResetRequested
OrderShipped
```

Notification platform converts business events to delivery jobs.

Avoid tightly coupling every product service directly to every provider.

## 3. Data Model

```text
NotificationRequest
- id
- tenant_id
- user_id
- type
- priority
- created_at
- idempotency_key
- status

DeliveryAttempt
- notification_id
- channel
- provider
- attempt_no
- provider_message_id
- status
- error
- next_retry_at

Preference
- user_id
- notification_type
- channel
- enabled/version
```

Templates stored/versioned separately.

## 4. Architecture

```text
Product Services
      ↓
Notification API/Event Ingest
      ↓
Durable Request DB / Outbox
      ↓
Router
  ┌───┼────┐
Critical  Transactional  Bulk
 Queue      Queue         Queue
  ↓           ↓             ↓
Workers by Channel/Provider
  ↓
Email / SMS / Push Providers
```

Separate priority/isolation is central to the change request.

## 5. Why Separate Queues

If one queue contains:
- 10M marketing jobs;
- one password reset at tail,

FIFO-like processing may delay critical work.

Use:
- separate queues;
- reserved worker capacity;
- provider quotas per class;
- admission control.

This is a **bulkhead**.

## 6. Preferences

At send time:
- fetch cached preference;
- ensure critical/legal semantics.

Some messages bypass marketing opt-out:
- password/security/service messages depending on product/legal requirements.

Do not invent policy; model notification type and preference rules explicitly.

Preference updates rare → cache useful.
For strict opt-out, use version/fresh lookup/invalidation.

## 7. Templates

Template:
```text
id/version/locale/channel/body
```

Render with validated variables.

Version templates so a retry can use intended content rather than silently changing mid-workflow.

Avoid injection:
- escape HTML;
- validate URL placeholders;
- secure template editor.

## 8. Provider Abstraction

```text
EmailWorker → ProviderAdapter
             ├─ Provider A
             └─ Provider B
```

Provider adapter normalizes:
- send;
- error categories;
- provider ID;
- webhook status.

Do not assume fallback provider guarantees delivery; sender reputation/config may differ.

## 9. Retries

Transient:
- timeout;
- 5xx;
- temporary quota.

Use backoff + jitter.

Permanent:
- invalid phone/email;
- opt-out;
- provider hard rejection.

Do not retry endlessly.

After bounded attempts:
- DLQ/failed state;
- alert depending on criticality.

## 10. Idempotency / Dedup

Upstream may retry API.
Use request idempotency key.

Worker may receive same delivery job twice.
Use unique logical delivery:
```text
UNIQUE(notification_id, channel)
```
or attempt state machine.

External provider:
- provider idempotency key if available.

Exactly-once end-to-end not assumed.

## 11. Provider Webhooks

Provider may later report:
```text
delivered
bounced
failed
```

Webhook:
- verify signature;
- idempotent event ID/provider message ID;
- update delivery state;
- duplicates/out-of-order possible.

Delivery status is eventual.

## 12. Rate Limits / Quotas

Providers impose limits:
```text
SMS 100/s
Email 10k/s
```

Workers need token bucket/admission control so queue buffers overflow rather than provider gets hammered.

Also:
- per tenant;
- per recipient;
- anti-spam.

## 13. Campaign Fan-Out

Campaign:
```text
campaign id + audience query
```

Do not create 10M jobs in one DB transaction.

Fan-out service:
- scan audience in pages;
- emit delivery jobs gradually;
- checkpoint;
- resumable;
- rate controlled.

Bulk campaign queue isolated from transactional notifications.

## 14. Failure Scenarios

### Provider A down
Circuit breaker; queue grows; optionally switch provider; preserve critical capacity.

### DB/outbox committed but publisher dies
Outbox publisher resumes.

### Worker sends then crashes before ack
Duplicate possible → idempotency/provider key.

### Preference service down
Use cached last-known preference only if safe; marketing may fail closed to avoid unwanted messages.

### Campaign creates huge backlog
Bulk queue only; critical queue unaffected.

## 15. Observability

Metrics:
- accepted notifications/sec;
- queue depth/oldest age by priority;
- provider latency/error;
- delivery rate;
- retry/DLQ;
- preference rejection;
- template errors;
- cost by channel/tenant.

SLO may differ:
- password reset time-to-provider;
- marketing completion within hours.

## 16. Multi-Region

Could ingest regionally but provider sends/global preference ownership need consistency.

Use:
- tenant/user home region;
- region-local queues;
- global campaign coordinator;
- failover rules.

Avoid duplicate send during region failover via durable idempotency/business keys.

## Interview Questions

1. How do you guarantee password resets beat marketing?
2. What if provider accepts message then times out?
3. How do you dedupe retries?
4. Why store notification status?
5. What if preferences update after job queued?
6. How do you send 100M campaign messages safely?
7. What if provider has 1k/s quota?
8. How do you process provider webhooks?

## Common Mistakes

- One queue for everything.
- Immediate provider calls from product services.
- Infinite retries.
- No idempotency.
- No preference model.
- Campaign fan-out in request transaction.
- No provider quotas/backpressure.
- No webhook signature/dedup.
- No cost awareness for SMS.
- “Fallback provider” with no duplicate/uncertain-send discussion.

## Must Remember

- **Separate critical and bulk workloads.**
- **Durably accept, then process asynchronously.**
- **At-least-once + idempotency is practical.**
- **Providers need timeout/retry/circuit-breaker/quota isolation.**
- **Preferences and templates are versioned business data.**
- **Campaign fan-out must be paged/resumable/backpressured.**
- **Delivery status is usually eventual.**
- **Queue age by priority is a key SLO signal.**

## Self-Score

Use the 40-point rubric and repair the two weakest areas.
