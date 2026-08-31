# Session 20 — Pub/Sub, Streams & Reprocessing

## 1. Must Learn

### Queue vs Pub/Sub
- **Understand:** Know a queue typically distributes tasks among workers; Pub/Sub lets independent subscribers react to one event.
- **Decision/trade-off:** Single logical task execution vs fan-out to multiple consumers.

### Event stream/log model
- **Understand:** Understand append-oriented retained events with topics, partitions, and offsets.
- **Decision/trade-off:** Replay/history vs storage/consumer-state complexity.

### Partitions & ordering
- **Understand:** Know ordering is typically guaranteed within a partition, and key choice affects parallelism/skew.
- **Decision/trade-off:** Per-key ordering vs throughput/load distribution.

### Consumer groups & offsets
- **Understand:** Understand group members share partition work and offsets track progress.
- **Decision/trade-off:** Scale-out consumption vs rebalance/lag management.

### Retention & replay
- **Understand:** Know historical events can be re-read for recovery, new consumers, or backfill.
- **Decision/trade-off:** Recoverability/flexibility vs duplicate side effects and storage.

### Consumer lag & skew
- **Understand:** Know lag means processing is behind; hot partitions limit group throughput.
- **Decision/trade-off:** Ordering/locality vs balanced load.

### Reprocessing/backfill safety
- **Understand:** Replay must handle duplicates, changed code/schema, and downstream side effects.
- **Decision/trade-off:** Correct rebuild/recovery vs accidental duplicate effects.

## 2. Should Know

- Batch vs stream processing conceptually.
- Aggregation windows conceptually.
- Log compaction conceptually as retaining latest keyed state.
- Consumer rebalancing conceptually.

## 3. Recognition Only

- Kafka internals/controllers
- Exactly-once stream-processing internals

## 4. Important Comparisons

- Queue vs Pub/Sub.
- Queue vs retained event stream.
- Single partition ordering vs many-partition throughput.
- Live processing vs replay/reprocessing.
- Batch vs stream processing.

## 5. Important Interview Questions

1. Is this a task to perform, or an event multiple systems should observe?
2. What ordering scope is required and what key determines partitioning?
3. What happens when a consumer falls behind?
4. How does a new consumer start from historical data?
5. What side effects could be duplicated during replay?
6. What happens when one partition becomes hot?

## 6. Common Interview Mistakes

- **Using Pub/Sub like a one-worker task queue without thought** → Choose based on fan-out vs work distribution.
- **Claiming global ordering at high scale** → Prefer partition-scoped ordering.
- **Replaying without idempotency** → Expect duplicate effects.
- **Ignoring consumer lag** → Treat lag as capacity/health signal.
- **Too few/many partitions without trade-off** → Partition count affects parallelism, ordering, and management.

## 7. Communication

### Important Vocabulary

Pub/Sub, event stream, topic, partition, offset, consumer group, retention, replay, consumer lag, partition skew, rebalancing, batching, backfill

### Useful Interview Phrases

- “This is an event multiple consumers need independently, so Pub/Sub fits better than a single task queue.”
- “Ordering is required per entity, so I’d partition by that key.”
- “Replay is useful only if consumers are safe against duplicate effects.”

### Important Questions to Ask the Interviewer

- **Question:** “Should multiple independent systems receive the same event?”  
  **Why it matters:** Determines Pub/Sub vs queue.
- **Question:** “What ordering scope matters?”  
  **Why it matters:** Determines partition key.
- **Question:** “Must we replay history for recovery/backfills?”  
  **Why it matters:** Determines retention/stream choice.

## 8. ⭐ Must Remember

1. Queue = distribute work; Pub/Sub = fan out events.
2. Streams retain events and enable replay.
3. Ordering is usually partition-scoped.
4. Consumer groups scale processing across partitions.
5. Lag measures how far consumers are behind.
6. Replay can duplicate side effects.
7. Partition skew can cap throughput.

## 9. Study Priority

1. Study first: queue vs Pub/Sub, topics/partitions/offsets.
2. Study next: consumer groups, ordering, retention/replay.
3. Finish with: lag, skew, reprocessing safety, batch vs stream.

## 10. Revision Checklist

- [ ] Choose queue vs Pub/Sub/stream.
- [ ] Explain partition ordering and key choice.
- [ ] Explain consumer groups/offsets.
- [ ] Handle consumer lag/hot partitions.
- [ ] Explain replay/backfill safety.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
