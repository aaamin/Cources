# Session 43 — Mock #3 — Data-Heavy / Write-Heavy System

**Phase:** Phase 4 — Timed Mock Interviews  
**Recommended time:** 45–55 minute mock + 30–45 minute review

## Session Goal

Test partitioning, ingestion, backpressure, storage/retention, and replay on an unseen write-heavy prompt.

## What You Need to Read / Learn

- Choose an unseen prompt such as IoT telemetry, audit events, ad-click aggregation, or distributed counters.
- Make partition key and skew explicit.
- Include durable ingestion/buffering only if the workload justifies it.
- Discuss retention and raw versus derived data.
- Discuss replay/reprocessing and idempotency.

## What You Need to Do

- [ ] Require yourself to state the partition key and one hotspot failure mode.
- [ ] Explain behavior when consumers fall behind.
- [ ] After the mock, redo the partitioning/backpressure section if either scored below 3.

## **Must Remember for the Interview**

- **High write throughput is usually an ingestion + partitioning + storage problem, not merely 'use Kafka'.**
- **Queue/stream backlog is bounded; backpressure and admission policy matter.**
- **Retention and cardinality can dominate cost.**
- **Replay is useful only if downstream processing can safely reprocess data.**
- **Skew can defeat a theoretically scalable partition scheme.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Ingress → partitioned durable buffer → consumers → raw/derived stores.**
- **State partition key, ordering scope, retention, replay.**
- **Handle hot tenants/keys and lag.**
- **Pass target = 32/40, no category below 2.**
- **Tie estimates to partition count/storage/backlog decisions.**

## Self-Test Before Marking This Session Complete

- [ ] Did I choose a partition key?
- [ ] Did I discuss backpressure/lag?
- [ ] Did I address raw vs derived retention?
- [ ] Did I describe replay correctness?
- [ ] Did estimates influence architecture?

## Completion Rule

Mark this session complete only after the timed mock, rubric score, and targeted repair notes. **Do not use the total score to hide a critical category weakness.**


## Session-Specific Notes

### 40-Point Rubric

Score 0–4 in each category:

1. Requirements & Scope
2. Estimation & Workload
3. APIs / Events / Data Model
4. High-Level Design & Flows
5. Scalability & Performance
6. Correctness & Consistency
7. Reliability & Operations
8. Security / Privacy / Cost
9. Trade-Offs & Evolution
10. Communication & Time Control

**Target:** 32/40 or higher, with no category below 2. A 0–1 in Requirements, APIs/Data, HLD, Correctness, Trade-offs, or Communication is a mock failure regardless of total score.


---

**Progress:** Session 43/46  
**Next:** Session 44
