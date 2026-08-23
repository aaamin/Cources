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
