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


## Important interview ideas

> **Important:** A stream is not just a faster queue. Its key feature is a **retained ordered history** that multiple independent consumers can replay.

### Events as facts

Prefer domain facts:

```text
OrderPlaced
PaymentCaptured
UserRegistered
```

over commands tightly coupled to one consumer:

```text
SendRowToAnalytics
```

Facts let new consumers appear later without changing the producer.

### Topic partitioning

A partition gives two properties:

1. a unit of parallelism;
2. an ordering scope.

If you key by `account_id`, all events for one account can stay ordered while accounts spread across partitions. A very hot account can still bottleneck one partition.

### Consumer groups

One logical application uses a consumer group. Partitions are assigned among group members, so adding consumers increases throughput only until each partition has an active consumer.

Separate applications use separate groups and independently track offsets.

### Replay and derived state

Retention allows:

- rebuilding a search index;
- recomputing an analytics view;
- fixing a bug and replaying events;
- onboarding a new consumer from history.

This is powerful, but replay can create huge downstream load. Consumers need idempotency and throttled catch-up.

### Event schema evolution

Retained events live longer than one deployment. New consumers may encounter old versions. Use explicit event versions or backward-compatible schema evolution. Avoid changing meaning of an existing field silently.

### Consumer lag

A consumer can be “healthy” but hours behind. Monitor both offset lag and **event-time lag**. If one partition is hot, adding more consumers does nothing until the key/partition strategy changes.

## Worked scenario — analytics + search

Order Service publishes `ProductUpdated` events. Search consumer updates the search index. Analytics consumer records change statistics. A new recommendation consumer can start later and replay retained events.

The Product DB remains source of truth. Search and analytics are derived and can be rebuilt if consumers fall behind or contain a bug.

## Interview questions and model answers

### Q1. “Queue or stream?”

I use a queue for background work where one logical consumer should perform each task. I use a retained stream when multiple independent consumers, replay, event history, or partitioned ordered processing are valuable. Some products blur the boundary; capabilities matter more than names.

### Q2. “What sets maximum consumer parallelism?”

Within one consumer group, the number of partitions. Extra consumers beyond partitions sit idle or cannot increase partition-level parallelism.

### Q3. “What happens if a consumer is buggy?”

Stop/fix it, reset to a safe offset or build a new consumer version, and replay retained events. Side effects must be idempotent or versioned to make replay safe.

### Q4. “Why does partition key matter?”

It controls both load distribution and ordering. Choose a key that preserves required per-entity order without concentrating too much traffic.

## Common mistakes to avoid

- Stream for everything because “Kafka scales.”
- No source of truth distinction.
- More consumers than partitions expected to increase throughput.
- Replay without idempotency.
- No event versioning.
- One hot partition ignored.
- Assuming global ordering across partitions.

## Short revision note

Stream mental model: **append event → partition by semantic key → retain → consumers track offsets → independent groups → replay derived state**.

## Topics to revise

- [ ] event vs command
- [ ] topic/partition
- [ ] offsets
- [ ] consumer groups
- [ ] retention/replay
- [ ] event schema versioning
- [ ] consumer lag
- [ ] hot partitions

## Interview-ready synthesis

### A strong 60–90 second explanation

I use streams when retained history, replay, independent consumers, and partitioned ordered processing matter. I choose a semantic partition key, explain consumer groups and offsets, monitor event-time lag, and keep event schemas backward compatible so history remains usable.

### How this topic connects to the wider system

- Scalability: partitions provide parallelism and ordering scope.
- Reliability: retention/replay rebuilds derived state after failures/bugs.
- Architecture: multiple consumers can react without producer coupling.
- Correctness: replay requires idempotent/version-aware consumers.

### Revision flashcards with answers

**Offset?**  
Consumer position within a partition's retained log.

**Consumer group?**  
One logical consuming application whose members split partitions.

**Can 100 consumers process 10 partitions faster?**  
Only up to partition-level parallelism in one group; extras do not help.

**Why event version?**  
Old retained events outlive deployments and must remain interpretable.

**Event-time lag?**  
How far processing is behind when events actually occurred.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Order events: analytics, notifications, fraud use independent consumer groups.
- Search indexing: replay retained product events to rebuild a broken index.
- Telemetry: partitioned stream absorbs high ingest volume and supports window aggregations.

### When not to overuse this idea

Do not introduce a retained event log for a tiny single-consumer background task when a simple queue/database job table is easier.

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

**Next:** Lesson 21 — Reliability, Overload & Failure Isolation
