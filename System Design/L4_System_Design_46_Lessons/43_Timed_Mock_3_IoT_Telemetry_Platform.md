# Session 43 — Timed Mock #3 — IoT Telemetry Platform

## Timed Mock Instructions

**Time:** 45–55 minutes  
**Notes:** None  
**Solution lookup:** Not allowed  
**Goal:** Lead the interview aloud while drawing.

### Prompt

> Design an IoT telemetry platform that ingests sensor events from millions of devices and supports recent dashboards plus historical analysis.

### Your responsibilities

You must:
1. clarify functional requirements;
2. prioritize non-functional requirements;
3. estimate only design-relevant workload;
4. define APIs/events and core data;
5. draw a simple high-level architecture;
6. narrate one main read and one main write;
7. deep-dive into the hardest requirement;
8. respond to interviewer changes;
9. handle one failure scenario;
10. close with trade-offs and 10× evolution.

### Focus

Write-heavy ingestion, partitioning, backpressure, replay/recovery, time-series storage, retention.

### Information You May Assume Initially

- Devices send timestamped measurements.
- Some devices send once/minute; others many times/second.
- Dashboard users query recent ranges by device/fleet/metric.
- Historical raw data must be retained more cheaply.
- Commands sent back to devices are out of scope.

---

# STOP SCROLLING

Start your timer now.

Do **not** open the sections below until the appropriate point in the mock.

<details>
<summary><strong>INTERVIEWER PACK — Open only after your initial architecture</strong></summary>

### Clarifications
- Assume millions of devices and hundreds of thousands to millions of events/sec at peak.
- Events may arrive late or out of order.
- Devices can reconnect and retry, causing duplicates.
- Some tenants own far more devices than others.

### Requirement Change
> A firmware bug causes one customer's devices to emit 20× normal traffic for two hours.

Ask:
- tenant quota?
- hot partitions?
- sampling/drop policy?
- other tenants protected?
- stream/storage backlog?

### Failure Scenario
> Time-series storage writes slow to one-third of ingest throughput for 45 minutes.

Ask:
- calculate backlog direction;
- stream retention/capacity;
- backpressure/admission;
- what gets dropped first if needed.

### Replay/Recovery Push
> A bug in hourly aggregation is discovered. Raw events for the last three days are still retained.

Ask how the candidate safely rebuilds aggregates without damaging the live pipeline.

</details>

<details>
<summary><strong>POST-MOCK REVIEW SIGNALS — Open only after the timer ends</strong></summary>

These are not a single canonical solution. Use them to detect whether you missed important reasoning.

Strong signals:

- device authentication and bounded batched ingest;
- durable stream separates ingest from storage/aggregation;
- partition key avoids one tenant/device hot spot (tenant + shard/device hash/time bucket);
- event ID/sequence for duplicate handling;
- late/out-of-order data explicitly accepted with window policy;
- raw object archive plus hot time-series store;
- retention/downsampling strategy;
- per-tenant quotas/fair scheduling;
- high-cardinality tag controls;
- stream lag/oldest time as key metrics;
- storage slowdown leads to lag, then backpressure/sampling/admission rather than infinite queue;
- replay writes versioned/idempotent aggregates and is throttled away from live work;
- multi-region local ingest may reduce latency/data-residency cost.

Critical warning signs:
- timestamp-only partition;
- “Kafka stores it until DB recovers” with no capacity math;
- global ordering;
- more consumers when storage is already saturated;
- no tenant isolation.


</details>

## 40-Point Scorecard

Score 0–4 in each category:

| Category | Score |
|---|---:|
| Requirements & Scope | /4 |
| Estimation & Workload | /4 |
| APIs / Events / Data Model | /4 |
| High-Level Design & Flows | /4 |
| Scalability & Performance | /4 |
| Correctness & Consistency | /4 |
| Reliability & Operations | /4 |
| Security / Privacy / Cost | /4 |
| Trade-Offs & Evolution | /4 |
| Communication & Time Control | /4 |
| **Total** | **/40** |

## Repair Rule

After scoring:

1. Identify the bottom two categories.
2. Write the single most damaging mistake in each.
3. Perform 2–3 narrow drills.
4. Redo only the weakest 15–20 minutes from a blank page.
5. Do not memorize a “perfect architecture.”

## Mock Completion Record

```text
Date:
Duration:
Score:
Bottom category #1:
Bottom category #2:
Best decision:
Biggest miss:
Requirement-change response:
Failure-scenario response:
What I will repair:
```
