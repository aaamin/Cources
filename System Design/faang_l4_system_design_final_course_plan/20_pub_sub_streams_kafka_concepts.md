# Session 20 — Pub/Sub, Streams & Kafka Concepts

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand event distribution and append-only logs well enough to choose queues versus streams and reason about partitioning/replay.

## What You Need to Read / Learn

- Pub/Sub: one published event can be consumed independently by multiple subscribers.
- Append-only event log/stream.
- Topics and partitions.
- Offsets and consumer progress.
- Consumer groups and parallel consumption.
- Retention and replay.
- Ordering within a partition.
- Partition-key choice and hotspots.
- Log compaction conceptually.
- Batch versus stream processing.
- Event-time/windowed aggregation at recognition depth.

## What You Need to Do

- [ ] Design click-event ingestion with three independent consumers: analytics, fraud, recommendations.
- [ ] Choose a partition key and explain ordering behavior.
- [ ] Explain how a new consumer rebuilds state by replaying retained events.

## **Must Remember for the Interview**

- **Pub/Sub is about fan-out to independent consumers; a work queue is often about assigning work to one worker group.**
- **Streams retain events, enabling replay and new consumers.**
- **Ordering is usually guaranteed only within a partition.**
- **Partition key determines both parallelism and ordering scope.**
- **Replay requires idempotent/replaceable downstream processing.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Topic → partitions → ordered records per partition → offsets.**
- **Consumer group shares partitions for parallelism.**
- **Retention enables replay/reprocessing.**
- **Choose partition key based on ordering + load distribution.**
- **Queue vs stream is a semantics decision, not a brand decision.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain queue vs Pub/Sub vs stream?
- [ ] Can I explain partition and offset?
- [ ] Can I explain consumer groups?
- [ ] Can I describe replay and its correctness risks?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 20/46  
**Next:** Session 21
