# Session 39 — Advanced Design — Metrics / Logging Platform

## Interview Prompt

> Design a multi-tenant metrics/logging ingestion platform. Clients emit high-volume telemetry. The system must ingest, retain, aggregate, and query data.

### Change Request

> One tenant suddenly produces 50× its normal traffic.

This is a write-heavy/backpressure/skew interview.

---

# STOP — Complete Your Design First

You should cover:
- agents/ingest;
- batching/compression;
- partitioning;
- durable stream;
- storage tiers;
- time-series indexing;
- aggregation;
- retention;
- query path;
- hot tenants;
- backpressure;
- replay.

---

# Interviewer Pressure Pack

### Pressure 1
A single tenant jumps from 100k events/s to 5M events/s.

### Pressure 2
The durable stream is healthy, but storage writers are 3× slower for 30 minutes.

### Pressure 3
A bug in aggregation logic is discovered after 12 hours.

### Pressure 4
Users create labels with random UUID values, exploding time-series cardinality.

Respond before continuing.

---

# Reference Reasoning

## 1. Separate Metrics and Logs Semantics

They can share pipeline ideas but differ in query/storage.

Metrics:
```text
metric_name
timestamp
value
labels/tags
```

Logs:
```text
timestamp
service
level
message
structured fields
```

Metrics often aggregate/downsample.
Logs often need text/filter search.

You can design common ingest + specialized storage/index.

## 2. Ingest

Agents should:
- batch;
- compress;
- retry bounded;
- buffer locally for short outages;
- include tenant identity;
- authenticate.

```text
Services/Agents
   ↓
Regional Ingest Gateways
   ↓
Durable Event Stream
```

Gateway should perform:
- auth;
- quota;
- schema/basic validation;
- tenant tagging;
- size limits.

Avoid heavy parsing/enrichment on critical ingest path if it risks drops.

## 3. Why a Durable Stream

Producer arrival can be bursty.

Stream buffers:
```text
ingest rate > storage write rate temporarily
```

It also enables:
- multiple consumers;
- replay;
- aggregation;
- raw archival.

But stream capacity is finite. Sustained overload still needs backpressure/admission.

## 4. Partition Key

Naive:
```text
partition by tenant_id
```

One huge tenant → one hot partition.

Better:
```text
hash(tenant_id, stream_shard)
```

where large tenant spreads across K shards.

Need query/reaggregation later across those shards.

Alternative:
- dedicated partitions/topic for huge tenant.

For metrics where ordering across all events is unnecessary, parallelism is valuable.

## 5. Ordering

Telemetry generally does not need global ordering.

Use:
- timestamp/event time;
- sequence where needed;
- partition-local processing.

Late/out-of-order data is normal.

Aggregation windows should allow lateness policy.

## 6. Batch / Compression

Agents send e.g. hundreds/thousands of events per request.

Benefits:
- less network overhead;
- compression ratio;
- fewer stream writes.

Trade:
- small delay;
- larger loss if local batch not durably buffered;
- max request limits.

## 7. Raw Storage

Raw telemetry can be archived in cheap object storage:

```text
tenant/date/hour/partition
```

Benefits:
- long retention;
- replay;
- offline analysis.

Hot query store retains recent/indexed data.

## 8. Metrics Storage

Time-series store optimized for:
- time range;
- metric/tag filtering;
- aggregates.

Partition dimensions might include:
```text
tenant
time bucket
metric hash/shard
```

Avoid one current-time shard receiving all writes.

## 9. Logs Storage

Possible:
- object storage for raw logs;
- search index for recent searchable window.

Search index is expensive.
Retain:
- 7–30 days searchable;
- older in object archive.

Exact window depends product/cost.

## 10. Aggregation

For metrics:
```text
raw 10-second data
→ 1-minute aggregate
→ 1-hour aggregate
```

Older data can use lower resolution.

Store:
- count;
- sum;
- min/max;
- quantile sketches conceptually if needed.

Do not try to average percentiles naively.

Recognition depth: p99 aggregation needs mergeable histogram/sketch, not average of p99s.

## 11. Query Path

```text
Dashboard
  ↓
Query API
  ↓
Query Planner
  ├→ recent TS/Search store
  └→ older object/aggregate store
```

Protect system from expensive user queries:
- max time range;
- result limits;
- tenant quotas;
- query concurrency;
- preaggregates/cache.

## 12. Hot Tenant Change

Tenant jumps 50×.

Problems:
- ingest gateways;
- stream partition;
- storage writers;
- index;
- query side.

Defenses:

### Per-tenant quota
Reject/sample/degrade beyond contracted limit.

### Partition tenant across shards
Avoid one hot partition.

### Separate dedicated resources
Very large tenants can get isolated pipeline.

### Fair scheduling
Do not let one tenant monopolize storage writers.

### Sampling
For noncritical metrics/logs, sample when overload—explicit contract.

### Priority
Security/audit logs may be more critical than debug logs.

## 13. Backpressure

Storage is 3× slower for 30 min.

Stream lag grows.

Calculate:
```text
backlog growth = ingest - consume
```

Actions:
- autoscale consumers if storage capacity available;
- reduce ingest/sample lower-priority;
- extend retention/disk capacity;
- alert on time lag;
- do not simply let lag grow indefinitely.

If storage is the bottleneck, more consumers can worsen it.

## 14. Replay

Aggregation bug discovered.

If raw events retained in stream/object store:
- deploy fixed version;
- reprocess affected window;
- write to versioned aggregate tables;
- compare;
- switch reads.

Do not overwrite live aggregates unsafely.

Replay needs:
- idempotent/versioned writes;
- resource isolation from live traffic;
- throttling.

## 15. Cardinality Explosion

Metric:

```text
http_requests{request_id="<random uuid>"}
```

Every request produces a unique time series.

This destroys index/memory.

Enforce:
- label allowlist;
- cardinality quotas;
- drop/hash dangerous tags;
- detect high-cardinality dimensions;
- documentation.

Tenant-aware guardrails are essential.

## 16. Retention

Example:
- raw metrics 7 days;
- 1-min aggregates 90 days;
- hourly 2 years;
- logs searchable 14 days;
- archive 1 year.

Lifecycle deletes old object/index data.

Retention is product/cost/privacy decision.

## 17. Availability vs Durability

Telemetry may accept some loss depending class.

Metrics:
- small drop may be tolerable.

Audit/security logs:
- durable delivery may be critical.

Classify streams rather than applying one guarantee universally.

## 18. Multi-Region

Ingest locally:
- reduce client latency/network;
- each region has stream/storage.
- global query fans out/uses centralized aggregates.

Tenant data residency may require staying regional.

Avoid shipping every raw byte globally unless requirement justifies cost.

## 19. Observability of Observability

Monitor:
- ingest accepted/dropped;
- bytes/sec;
- per-tenant quota violations;
- partition skew;
- stream lag time;
- storage flush latency;
- index backlog;
- query p95/p99;
- cardinality;
- retention job health.

## Common Mistakes

- Tenant ID alone as partition key.
- “Kafka buffers it” with no sustained overload plan.
- More consumers while storage is saturated.
- No high-cardinality defense.
- Search-index every log forever.
- No raw archive/replay.
- Global ordering.
- No query guardrails.
- No per-tenant fairness/quotas.
- Metrics and audit logs treated with same loss policy.

## Must Remember

- **Ingest, buffer, store, aggregate, query are separate stages.**
- **Durable stream absorbs bursts but not infinite sustained overload.**
- **Partitioning must survive hot tenants.**
- **Backpressure/admission protects storage.**
- **Raw archive enables replay/rebuild.**
- **High-cardinality labels can destroy metrics systems.**
- **Retention/downsampling control cost.**
- **Query workloads need their own limits.**
- **Telemetry durability can differ by data class.**

## Repair Exercise

Design only the overload path:

> Tenant A produces 5M events/s for one hour. Storage can accept only 2M/s for that tenant. Other 1,000 tenants must stay healthy.

Specify quotas, buffering, sampling/drop policy, partitioning, and what metrics trigger intervention.
