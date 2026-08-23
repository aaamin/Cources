# Lesson 19 — Message Queues & Async Processing

**Phase:** Fundamentals  
**Session:** 19/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn how queues decouple producers from workers, how acknowledgements and retries work, why duplicates are normal, and how backlogs expose overload.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Why asynchronous processing

Not every task must finish before the user receives a response. Background work such as image resizing, email delivery, report generation, or feed fan-out can move to a queue. This shortens synchronous latency and isolates slow downstream systems.

## Producer, broker, consumer

The producer submits a work item, a broker stores/routes it, and workers consume it. The queue temporarily absorbs bursts when arrival rate is higher than worker processing rate, as long as the backlog stays within acceptable delay and storage limits.

## Acknowledgements and visibility

A worker typically acknowledges a job after it successfully finishes. If the worker crashes first, the broker makes the job visible again after a timeout/lease. This creates at-least-once processing, so duplicate execution is expected.

## Delivery semantics

At-most-once may lose work but avoids retry duplicates. At-least-once retries work but can repeat it. “Exactly once” at the business level usually comes from idempotent state transitions or transactions—not from assuming the transport will never redeliver.

## Retries and DLQ

Transient failures retry with exponential backoff. Permanently invalid/poison messages should eventually move to a dead-letter queue so they do not loop forever. A real system needs an operational process for inspecting, fixing, and safely replaying them.

## Backpressure and backlog

If producers create 10k jobs/s but workers finish 1k/s, queue depth grows forever. Useful signals include depth, oldest-message age, consume rate, retry rate, and DLQ count. Adding workers only helps if downstream dependencies can absorb the extra load.

## Worked example — image thumbnail processing

The upload API stores the original object and enqueues `GenerateThumbnail(image_id)`. A worker writes to a deterministic output key and acknowledges afterward. If it crashes after writing but before ack, the job repeats but overwrites the same object, making the effect idempotent.

## Interview lens

Whenever you add a queue, explain what left the synchronous path, retry behavior, duplicate handling, ordering needs, and what backlog means operationally.

## What to remember

Queues provide decoupling and burst absorption, but introduce delivery semantics, retries, duplicates, poison work, and backlog management.

## Review after reading

1. Why queue work?
2. Why does at-least-once duplicate?
3. What is a DLQ?
4. Which metric shows growing lag?
5. Why should workers be idempotent?

## Deeper study notes

### Queue depth is stored latency

A queue with one million messages may be healthy if workers drain quickly, or disastrous if the oldest item is hours old. Monitor both depth and **age of oldest message**. User impact is related to waiting time, not only count.

### Ordering reduces parallelism

If all messages require one global order, one logical sequence becomes a bottleneck. Most applications only need ordering by a key: one user, account, or conversation. Partition by that key so unrelated work can proceed in parallel.

### Poison messages need isolation

A malformed job that always crashes a worker can consume infinite retries. Retry policies should distinguish transient vs permanent failure and move persistent failures to a DLQ with enough metadata to debug.

### Queue is not infinite shock absorber

Brokers have storage and retention limits. A persistent producer/consumer mismatch eventually exhausts capacity. Admission control or upstream throttling is required if the backlog cannot recover within the product's delay budget.

### Common mistakes

- Acknowledging before the durable side effect completes when loss is unacceptable.
- Assuming retry means exactly once.
- Scaling consumers without checking DB/provider limits.
- Requiring global ordering unnecessarily.
- Forgetting DLQ replay itself can create a second traffic spike.


## Important interview ideas

> **Important:** Queues move time and failure boundaries. They let producers finish before consumers, but now you must design backlog, delivery semantics, retries, duplicates, and poison messages.

### What a queue buys

A queue provides **temporal decoupling**. Producer and consumer do not need to run at the same speed at the same moment. This is valuable for bursty traffic and external dependencies.

Example:

```text
API accepts 10k image uploads/min
workers can resize 8k/min
```

The queue absorbs the short burst. But if the mismatch continues forever, backlog grows forever. A queue is not infinite capacity.

### Delivery lifecycle

A common model:

1. broker makes message visible;
2. worker receives/leases it;
3. worker performs side effect;
4. worker acknowledges;
5. broker removes/advances message.

If the worker dies between steps 3 and 4, the side effect happened but the message will be retried. This is why idempotency is fundamental.

### Ordering scopes

Global order destroys parallelism. Usually you need only per-key ordering:

```text
conversation_id
account_id
order_id
```

Partition the queue/stream by that key. Different keys process concurrently.

### Retry classification

Not every error should retry:

- timeout / 503 → probably transient;
- invalid email address → permanent;
- authentication/config error → operator action;
- rate limit → retry after provider window;
- corrupt payload → DLQ.

Retries should have backoff and a maximum attempt policy.

### DLQ operations

A dead-letter queue is not the end of the design. Define:

- alert threshold;
- what metadata is retained;
- how operators diagnose;
- whether replay is safe/idempotent;
- how to avoid replaying millions at once.

## Worked scenario — payment receipt email

Order Service commits an order and emits `OrderConfirmed`. Notification worker consumes it and calls email provider.

If provider times out after sending, the worker may retry. To reduce duplicates, assign a stable notification ID and use provider idempotency if supported. If the email address is malformed, do not retry forever; move to DLQ and record delivery failure.

## Interview questions and model answers

### Q1. “At-most-once vs at-least-once?”

At-most-once avoids redelivery but can lose work. At-least-once retries unacknowledged work and can duplicate it. Most durable business workflows use at-least-once plus idempotent consumers because losing work is worse than handling duplicates.

### Q2. “What happens if queue depth grows?”

Latency grows because messages wait. I monitor depth and oldest-message age. Scale consumers only if downstream systems have capacity; otherwise apply admission control, priority isolation, or reduce producer rate.

### Q3. “Why not acknowledge immediately on receive?”

If the worker crashes after acknowledgement but before the side effect, the message is lost. Acknowledge after the required durable effect when loss is unacceptable.

### Q4. “How do you preserve order?”

Define the smallest ordering scope. Partition messages by that key and process each partition in order. Avoid global ordering unless the product genuinely needs it.

## Common mistakes to avoid

- “Queue means exactly once.”
- No idempotency story.
- Unlimited retries.
- No DLQ/replay process.
- Scaling consumers beyond database/provider capacity.
- Global ordering for unrelated work.
- Treating queue depth as harmless.

## Short revision note

Queue checklist: **producer → durable enqueue → lease/ack → retry/backoff → idempotent consumer → DLQ → backlog metrics → ordering scope**.

## Topics to revise

- [ ] producer/consumer/broker
- [ ] ack / visibility timeout
- [ ] at-most vs at-least once
- [ ] idempotent workers
- [ ] retry/backoff
- [ ] DLQ
- [ ] queue depth/age
- [ ] ordering scope

## Interview-ready synthesis

### A strong 60–90 second explanation

I add queues when work can be asynchronous or bursty. I define when the message is acknowledged, what delivery guarantee applies, how retries/backoff and DLQ work, what ordering scope is required, and why consumers are idempotent. I monitor queue age, not only queue length.

### How this topic connects to the wider system

- Reliability: durable queue prevents transient worker/provider failure from losing accepted work.
- Performance: asynchronous processing shortens user critical path and absorbs bursts.
- Correctness: at-least-once means duplicate effects must be controlled.
- Scalability: partitioned queues/workers increase throughput until downstream becomes bottleneck.

### Revision flashcards with answers

**Visibility timeout?**  
A lease period after which unacknowledged work can become available for retry.

**Poison message?**  
A permanently bad item that repeatedly fails and should be isolated/DLQ.

**Queue age?**  
How long oldest work has waited; directly represents processing delay.

**Ordering scope?**  
Smallest key whose events need order, e.g. one conversation/account.

**Why not acknowledge early?**  
Crash after ack before effect would lose work.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Notifications: provider calls are queued so provider latency/outage does not block business APIs.
- Video: transcode tasks retry independently after upload.
- Crawler: fetch tasks use leases so crashed workers' URLs become available again.

### When not to overuse this idea

Do not queue an operation when the caller genuinely needs its immediate result and there is no meaningful asynchronous boundary.

### A good interviewer sentence

> “I would use this only because the current requirement/workload creates the specific problem it solves. If that assumption changes, I would simplify or choose the alternative.”

This sentence captures an important L4 behavior: architecture is conditional, not dogmatic.

## Personal notes

```text
Concepts that are clear:

Concepts to revisit:

Three things to remember:
1.
2.
3.

Questions for later:
```

---

**Next:** Lesson 20 — Pub/Sub, Streams & Kafka Concepts
