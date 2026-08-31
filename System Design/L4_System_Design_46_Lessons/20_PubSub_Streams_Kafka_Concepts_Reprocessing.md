# Session 20 — PubSub, Streams, Kafka Concepts & Reprocessing

## Outcome

You should understand when to use work queues, Pub/Sub, or durable streams; explain topics, partitions, offsets, consumer groups, retention, replay, partition ordering, consumer lag, batching, aggregation windows, reprocessing, and the operational consequences of replay.

## Queue vs Pub/Sub vs Stream

These ideas overlap in products, but use the mental distinction.

### Work queue

```text
Task → one worker performs it
```

Example:
- resize image;
- send email;
- generate report.

### Pub/Sub

```text
Event
 ├→ Analytics subscriber
 ├→ Email subscriber
 └→ Search subscriber
```

Each independent subscriber receives the event.

### Durable event stream/log

```text
ordered append log
offset 100
offset 101
offset 102
```

Consumers track position and can replay retained history.

Good for:
- event pipelines;
- analytics;
- CDC;
- durable integration;
- high-volume telemetry.

## Event vs Command

Useful distinction:

Command:
> “Send this email.”

Event:
> “OrderCompleted.”

A command usually has intended handler/work.
An event states a fact and can have many consumers.

Do not make every queue message an “event” without semantics.

## Topic

Logical stream/category:

```text
orders
payments
clicks
metrics
```

## Partition

A topic is split for parallelism.

```text
orders
 ├─ partition 0
 ├─ partition 1
 └─ partition 2
```

A record key determines partition in common designs.

Partition gives:
- scalable writes;
- consumer parallelism;
- ordering within partition.

## Ordering

Common stream guarantee:
- order within one partition;
- no simple global order across all partitions.

If all events for an `order_id` use same key:
- they land on same partition;
- per-order order can be preserved.

If you require global order:
- one partition is simplest but limits throughput/parallelism;
- global ordering systems are more expensive.

Most business requirements need per-entity ordering, not global.

## Offset

Position in partition log.

Consumer tracks:
```text
partition 3 → offset 582910
```

After processing, it commits/progresses offset.

If commit happens before side effect → possible loss.
If after side effect → possible duplicate on crash.

Same fundamental delivery issue as queues.

## Consumer Group

Consumers in one group share partitions.

Example:
```text
6 partitions
3 consumers
```
each handles ~2 partitions.

Add consumers up to partition count for parallel processing.

If 100 consumers but only 6 partitions, most cannot independently consume partitions in same group.

Different groups each get their own view:
- search group;
- analytics group;
- notification group.

## Retention

Stream keeps events for a configured duration/size rather than deleting immediately after one consumer.

Benefits:
- replay;
- new consumer bootstrap;
- reprocessing;
- audit/debug.

Costs:
- storage;
- privacy/retention implications;
- replay side effects.

## Replay

Move consumer offset backward or create new consumer to process old data.

Useful:
- bug fix;
- rebuild search index;
- recompute analytics;
- recover failed consumer.

But replay can repeat side effects.

Danger:
```text
replay OrderCompleted
   ↓
send customer email again
```

Therefore consumers need:
- idempotency;
- replay-aware behavior;
- separate derived-model rebuild modes;
- event versioning.

## Reprocessing / Backfill

Suppose a new field is needed in analytics.

Options:
- replay retained historical events;
- scan source DB and publish backfill;
- batch job.

Need:
- version/schema compatibility;
- throttling;
- isolated resources;
- avoiding impact on live traffic;
- duplicate handling.

## Consumer Lag

Lag = distance between newest available event and consumer progress.

If producer writes faster than consumer:
```text
lag ↑
```

Monitor:
- records behind;
- time behind;
- processing rate.

Lag is the stream equivalent of queue backlog.

## Partition Skew

If partition key is `tenant_id` and one tenant creates 50× traffic:
- one partition becomes hot;
- consumers for other partitions idle.

Solutions:
- better key/bucketing;
- split hot tenant;
- isolate tenant;
- scale partition count before peak;
- allow out-of-order across subkeys if semantics permit.

## Rebalancing Consumers

When consumers join/leave, partitions are reassigned.

During rebalance:
- processing can pause;
- in-flight work may be repeated;
- ownership changes.

Need idempotent processing and careful long-running tasks.

## Batching

Consume/process multiple events together.

Benefits:
- fewer network calls;
- efficient DB writes;
- compression;
- throughput.

Trade-offs:
- added latency while batch fills;
- one bad record handling;
- bigger retry unit unless per-record tracking.

## Aggregation Windows

For metrics/clicks:
- tumbling 1-minute window;
- sliding window;
- session-like grouping.

You do not need stream-processing framework internals. Understand that aggregates have:
- event time vs processing time questions;
- late data;
- window close/update policy.

## Log Compaction Concept

Some logs can retain latest record per key over time.

Useful for:
- current configuration/state reconstruction;
- compacted changelog.

Not the same as time retention.

Recognition level is sufficient.

## Schema Evolution

Events live longer than code deployments.

Consumers must handle:
- old event version;
- new optional fields;
- compatible schema changes.

Avoid destructive event contract changes that break replay.

## Worked Example — Order Event Stream

Order service writes order and outbox transactionally.

CDC/publisher emits:
```text
OrderCreated
OrderPaid
OrderCancelled
```

Key = `order_id`.

Consumers:
- analytics group;
- search/reporting group;
- notification group.

All events for same order preserve partition order.

If analytics bug discovered:
- deploy fixed consumer;
- replay from earlier offset;
- write derived analytics idempotently.

Notification consumer should not resend historical emails blindly during replay.

## Small Design Drills

1. Why do partitions enable scale?
2. Why can 100 consumers not fully parallelize a 10-partition topic within one group?
3. What is consumer lag?
4. Why is replay both powerful and dangerous?
5. Why key order events by order_id?
6. One tenant dominates one partition. What problem is this?
7. What is the difference between two consumer groups and two consumers in the same group?

<details>
<summary>Answer key</summary>

1. Writes/reads are distributed and partitions can be processed in parallel.
2. A partition is normally owned by one group consumer at a time; extra consumers idle.
3. Consumer's progress behind latest produced data, in records/time.
4. It rebuilds/reprocesses history but repeats events and may repeat side effects.
5. Keeps one order's events in one partition for per-order ordering.
6. Partition skew/hot partition.
7. Same group shares work; different groups independently consume the stream.

</details>

## Common Interview Mistakes

- Using Kafka as a task queue with no reason.
- Assuming global ordering.
- Forgetting partition count limits group parallelism.
- Replaying events without idempotency.
- Ignoring consumer lag.
- Partitioning by a low-cardinality/hot key.
- Treating committed offset as proof external side effect happened exactly once.
- Breaking old consumers with event schema changes.

## Must Remember

- **Queue = perform work; Pub/Sub = distribute event; stream = durable ordered history with replay.**
- **Partitions create parallelism and define ordering scope.**
- **Offsets track consumer position.**
- **Consumer groups share partitions.**
- **Retention enables replay.**
- **Replay repeats events and can repeat side effects.**
- **Consumer lag is a core health metric.**
- **Hot keys create partition skew.**
- **Batching improves throughput but adds latency/complexity.**
- **Event schemas must evolve compatibly.**

## Interview Revision Summary

Stream checklist:

```text
Why stream instead of queue?
Topic?
Partition key?
Ordering scope?
Partition count?
Consumer groups?
Offset commit semantics?
Retention?
Replay?
Idempotency?
Lag?
Skew?
Schema evolution?
```

## Explain Without Notes

Design an event stream for order changes used independently by analytics, search indexing, and notifications. Explain keying, ordering, replay, and why notifications need special replay handling.

## Completion Checklist

- [ ] I distinguish queue/PubSub/stream.
- [ ] I understand partitions/offsets/groups.
- [ ] I understand per-partition ordering.
- [ ] I monitor lag/skew.
- [ ] I design replay safely.
- [ ] I understand batching/schema evolution.
