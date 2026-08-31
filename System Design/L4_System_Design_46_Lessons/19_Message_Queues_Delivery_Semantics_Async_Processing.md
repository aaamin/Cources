# Session 19 — Message Queues, Delivery Semantics & Async Processing

## Outcome

You should be able to decide which work belongs off the synchronous request path, explain producer/queue/consumer mechanics, acknowledgements and visibility, retries/DLQs, at-most-once vs at-least-once, duplicate handling, idempotent consumers, ordering scope, backlog, and worker scaling.

## Why Asynchronous Processing Exists

A user request should usually wait only for work required to produce the immediate result.

Bad synchronous path:

```text
Checkout
  ↓
Save order
  ↓
Charge payment
  ↓
Send email
  ↓
Update analytics
  ↓
Update loyalty
  ↓
Return response
```

If email is slow, checkout becomes slow.

Better:

```text
Checkout → durable order result → response
                      |
                   events/jobs
                      ↓
              background workers
```

Move optional/retriable work to async processing.

## Queue Mental Model

```text
Producer → Queue → Consumer/Worker
```

Producer creates a message.
Queue stores/buffers it.
Consumer processes it.

The queue decouples producer speed from consumer speed—within finite storage/backlog limits.

## Typical Uses

- email/SMS/push;
- image/video processing;
- report generation;
- webhook delivery;
- background cleanup;
- asynchronous integration;
- work smoothing during bursts.

Do not queue work that requires an immediate synchronous answer unless the product supports an async job model.

## Acknowledgements

A consumer should acknowledge after work is safely complete.

Conceptual flow:

```text
receive message
   ↓
process
   ↓
persist side effect
   ↓
acknowledge
```

If consumer crashes before ack, broker may redeliver.

This is why duplicate processing must be expected.

## Visibility Timeout

In systems such as SQS-style queues, receiving a message temporarily hides it from other consumers.

```text
Consumer A receives
message invisible for 30 sec
```

If A finishes and deletes/acks, done.

If A crashes and timeout expires, message becomes visible again.

If processing takes longer than visibility:
- extend/renew visibility;
- choose appropriate timeout;
- make processing idempotent.

Otherwise another consumer can process the same message concurrently.

## Delivery Semantics

### At-most-once

Message is processed zero or one time.

Possible approach:
- remove/ack before processing.

Advantage:
- no duplicate processing.

Risk:
- work can be lost if consumer crashes after ack but before work completes.

Use when losing occasional event is acceptable but duplicates are harmful/expensive.

### At-least-once

Message is processed one or more times.

Ack after successful processing.

Advantage:
- durable work not silently lost.

Cost:
- duplicates are possible.

Common default for business workflows combined with idempotency.

### Exactly-once

End-to-end exactly-once is difficult because broker delivery is only one part.

Suppose worker:
1. charges credit card;
2. crashes before broker acknowledgement.

Broker redelivers.

Even if broker has “exactly-once” features internally, the external payment side effect may occur twice unless payment operation/idempotency handles it.

Therefore a strong practical statement is:

> I assume at-least-once delivery and make the consumer/side effect idempotent.

## Idempotent Consumer

Processing the same logical message multiple times produces the same intended business result.

Pattern:

```text
message has event_id
consumer DB has processed_event(event_id UNIQUE)
```

Transaction:
```text
if event not processed:
    apply side effect
    record event_id
```

For external APIs, use provider idempotency key if supported.

Never rely only on “duplicates are rare.”

## Retry

Transient failures can be retried.

Use:
- bounded retries;
- exponential backoff;
- jitter;
- delay queues when appropriate.

Do not immediate-loop retries against a failing dependency.

## Dead-Letter Queue

After repeated failure:

```text
main queue
   ↓ retries exhausted
DLQ
```

DLQ lets you:
- inspect poison messages;
- alert;
- fix/replay;
- avoid blocking healthy workload.

A DLQ is not a trash bin. It needs monitoring and operational process.

## Poison Messages

A specific malformed or permanently invalid message fails every attempt.

Without limits, it can retry forever.

Handle:
- validation;
- retry classification;
- DLQ;
- alert.

## Ordering

Queues may offer varying ordering guarantees.

Important question:
**ordering of what scope?**

Global ordering:
- expensive;
- limits parallelism.

Per-key ordering:
```text
conversation_id
account_id
order_id
```
allows many keys to process in parallel while maintaining order within a key/partition.

Even with ordered delivery, retries/duplicate side effects may complicate application order.

## Queue Backlog

If producer > consumer throughput:

```text
queue depth ↑
oldest message age ↑
```

A queue absorbs bursts, not infinite sustained overload.

Monitor:
- queue depth;
- oldest message age;
- processing rate;
- failure rate;
- DLQ size.

## Scaling Workers

If messages are independent:
- add consumers/workers.

Limits:
- broker partition count;
- downstream DB/API capacity;
- provider quotas;
- ordering constraints.

Adding 100 workers can DDoS your own database or email provider.

Use bounded concurrency/backpressure.

## Priority

Emergency password reset should not wait behind 10M marketing emails.

Options:
- separate queues;
- priority queue;
- reserved worker capacity.

Separate queues often give clearer isolation.

## Worked Example — Email Notifications

Request:
```text
POST /reset-password
```

Application:
1. generate secure reset token;
2. persist/reset state;
3. enqueue `PasswordResetEmailRequested`;
4. respond.

Worker:
1. receive;
2. check idempotency/event ID;
3. call email provider with timeout/idempotency where possible;
4. record result;
5. ack.

Failure:
- transient provider error → backoff retry;
- permanent bad address → mark failure/no endless retry;
- repeated unknown failure → DLQ.

Marketing campaign uses separate queue/provider quota so it cannot starve password resets.

## Small Design Drills

1. Worker sends email successfully then crashes before ack. What happens under at-least-once?
2. Why doesn't a queue create infinite throughput?
3. What metric tells you users' async jobs are getting older even if depth is stable?
4. Why can increasing workers hurt the system?
5. When might per-key ordering be enough instead of global ordering?
6. What should happen to a malformed message that always fails?

<details>
<summary>Answer key</summary>

1. Message may be redelivered; consumer/provider handling should prevent harmful duplicate side effect.
2. Sustained production above processing causes unbounded backlog until storage/latency limits fail.
3. Oldest-message age / queue delay.
4. Downstream DB/provider/broker can be overwhelmed; contention increases.
5. Chat messages per conversation, account ledger per account, order events per order.
6. Validate/classify, stop bounded retries, route to DLQ/inspection.

</details>

## Common Interview Mistakes

- Saying “queue guarantees exactly once.”
- Acking before durable side effect without discussing loss.
- Retrying forever.
- DLQ with no monitoring.
- Scaling consumers without downstream limits.
- Assuming ordering is global.
- Mixing critical and bulk work in one unbounded queue.
- Forgetting oldest-message age.
- Treating queue as source of truth when job state needs durable tracking.

## Must Remember

- **Async work leaves the request path when immediate completion is unnecessary.**
- **Ack after safe completion for at-least-once semantics.**
- **At-least-once implies duplicates.**
- **Idempotency is an application/business property.**
- **End-to-end exactly-once is difficult.**
- **Retries need backoff/jitter and limits.**
- **DLQ needs operations, not abandonment.**
- **Ordering should usually be scoped per key.**
- **Queue depth and oldest-message age reveal overload.**
- **Worker concurrency must respect downstream capacity.**

## Interview Revision Summary

Queue checklist:

```text
What is async?
Message schema/ID?
Durable?
Ack when?
At-most / at-least?
Idempotency?
Retry policy?
DLQ?
Ordering scope?
Backlog metric?
Consumer scale?
Downstream limit?
Priority/isolation?
```

## Explain Without Notes

Design asynchronous password-reset email processing that tolerates consumer crashes and a temporary provider outage without sending dangerous duplicates or blocking the API.

## Completion Checklist

- [ ] I can explain queue mechanics.
- [ ] I understand ack/visibility.
- [ ] I compare at-most and at-least-once.
- [ ] I design idempotent consumers.
- [ ] I understand retries/DLQ.
- [ ] I reason about ordering/backlog/scaling.
