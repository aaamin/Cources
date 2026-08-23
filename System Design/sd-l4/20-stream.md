# Lesson 20 — Pub/Sub, Streams & Kafka Concepts

**Phase:** Fundamentals  
**Session:** 20/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn how Pub/Sub differs from work queues and how durable event streams use topics, partitions, offsets, consumer groups, retention, and replay.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Queue vs Pub/Sub

A work queue usually means “this unit of work should be handled.” Workers compete for jobs. Pub/Sub means “this event occurred,” and multiple independent subscribers may each need a copy—analytics, notification, fraud, audit, etc.

## Append-only log

Streaming platforms model a topic as a retained append-only log, often split into partitions. Consumers maintain offsets showing their position. Because records remain for a retention period, consumers can replay historical events.

## Partitions and ordering

Ordering is usually guaranteed only inside one partition. The partition key therefore defines ordering scope. Keying chat events by conversation can preserve per-conversation order while processing different conversations in parallel.

## Consumer groups

Consumers in the same group divide partitions to scale one logical application. Different groups independently consume the same topic, which is why one event can drive analytics and notifications without each producer knowing those consumers.

## Retention and replay

Retained events let a repaired consumer reset offsets and rebuild state. Replay is useful for bug fixes, backfills, and launching a new derived view without making the source service resend old data.

## Batch vs streaming

Batch processing handles bounded datasets periodically; streaming updates continuously or in windows. Many systems combine both: real-time approximate aggregates plus exact offline recomputation.

## Worked example — order event fan-out

The Order Service publishes `OrderPlaced`. Notification, Analytics, and Fraud each use a separate consumer group. If Analytics deploys a bug, it can reset its offset and replay retained events while other groups continue normally.

## Interview lens

Use a stream when retention, replay, independent consumers, high throughput, or partitioned ordering matter. Use a simpler queue when you mainly need background work distribution.

## What to remember

Streams are durable histories: partition, offset, consumer group, retention, replay, and partition-scoped ordering are the core concepts.

## Review after reading

1. Queue vs Pub/Sub?
2. What is an offset?
3. Ordering scope?
4. Why consumer groups?
5. Why replay?

## Deeper study notes

### Events should describe facts, not commands, when possible

`OrderPlaced` says a durable business fact occurred. `SendAnalyticsRecord` couples the producer to a specific consumer action. Event-driven designs stay more extensible when producers publish domain facts and consumers decide how to react.

### Partition-key choice is semantic

If you need all events for one account in order, key by account ID. If throughput matters more than per-account order, choose a more distributed key. A partition is both a scalability unit and an ordering unit.

### Replay changes how you evolve systems

A new consumer can build its state from retained history. This is powerful for search indexing, analytics, and materialized views. But event schemas must remain interpretable over time, so versioning and backward compatibility matter.

### Consumer lag is a first-class metric

A consumer can be “up” but hours behind. Track lag in offsets and, more meaningfully, event-time delay. Autoscaling may help until a hot partition or slow downstream store becomes the true bottleneck.

### Common mistakes

- Using a stream for simple task dispatch without needing retention/replay.
- Assuming more consumers than partitions increases parallelism in one group.
- Repartitioning without considering ordering semantics.
- Publishing huge payloads when an object reference would be cheaper/safer.
- Breaking old consumers with incompatible event schema changes.


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

**Next:** Lesson 21 — Reliability, Overload & Failure Isolation
