# Lesson 39 — Design a Metrics / Logging Platform

**Phase:** Advanced Design  
**Session:** 39/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Ingest very high write volume.
- Retain data by policy.
- Support recent queries and aggregates.
- Isolate noisy tenants.
- Replay/reprocess after consumer bugs.

## 2. Scale and workload shape

Estimate events/sec, average bytes/event, compression, retention, cardinality, tenant skew, query QPS, and acceptable ingestion lag. Storage and ingestion throughput are usually dominant; high-cardinality indexing can dominate memory/cost.

## 3. API / contract surface

```http
POST /v1/ingest
GET  /v1/query?... 
```

Real agents should batch many events per request and compress them to reduce overhead.

## 4. Data model

Raw event fields:

```text
tenant_id
timestamp
metric/log body
tags/dimensions
```

Derived aggregate:

```text
(tenant, metric, time_bucket, dimension_set) → count/sum/min/max/...
```


## 5. High-level architecture

```text
Agents → Ingest LB → Durable Stream
                         ├─ Raw/Object Storage
                         ├─ Aggregators → Time-Series Store
                         └─ Indexers → Search Store

Query API → recent/aggregate/search stores
```

The durable ingest path should not depend on expensive interactive query indexes being healthy.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Partitioning

Partition by tenant + metric/hash while preventing one giant tenant from owning a single hot partition. Large tenants may receive dedicated subpartitions or quotas.

### Aggregation

Precompute minute/hour rollups for long-range queries. Keep recent raw high-resolution data for debugging, then downsample/expire according to retention policy.

### Cardinality

Tags such as `user_id` can create millions of unique series. Limit/charge high-cardinality dimensions because they explode index and memory cost.

### Replay

Retain raw events or stream long enough to rebuild aggregates/indexes after a bug. Write new corrected output versions before switching readers.

## 7. Failure modes and recovery

- Consumer lag: monitor lag/oldest event; scale consumers without overwhelming storage.
- Hot tenant: quota, shard further, or isolate resources.
- Search store down: keep ingesting into stream/raw storage.
- Aggregator bug: replay into new version.
- Ingest spike: buffer in stream but enforce finite admission/backpressure.
- Data corruption: raw immutable tier supports rebuild/reconciliation.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Separating durable ingestion from derived query stores improves resilience. More indexes improve flexibility but increase write amplification, cardinality cost, and operational complexity.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Ingest should acknowledge only after durable buffering

An agent can send a compressed batch. The ingest tier validates tenant/quota and appends to a durable stream/log. Once the broker's durability threshold is met, the agent can receive success. Expensive parsing/indexing happens later.

### Raw vs derived storage

Keep raw or minimally transformed data in cheap durable storage for the period needed to replay. Query-optimized time-series/search stores are derived views. If an indexing bug occurs, rebuild them rather than treating the corrupted index as authoritative.

### Time partitioning

Time-series data is naturally queried by time ranges, but partitioning only by timestamp hotspots current writes. Combine tenant/metric hashing with time buckets, for example `(tenant_shard, day)`, and maintain routing metadata as needed.

### High cardinality

A metric such as `http_requests{user_id=...}` creates one series per user and can make memory/index cost explode. Limit label cardinality, aggregate earlier, or send user-level detail to logs/traces instead of metrics.

### Query tiers

Recent data may live in a fast store; older high-resolution data can move to object storage; long-range queries use downsampled hourly/daily aggregates. This controls cost without deleting all historical insight.

### Common interview mistakes

- Synchronous search indexing before acknowledging ingest.
- One partition per tenant when one tenant can be gigantic.
- Ignoring retention/downsampling cost.
- Allowing unlimited user-defined tag cardinality.
- Scaling consumers without checking write capacity of their target DB.
- No replay path for fixing derived data.

### Reusable patterns learned

Durable ingestion log, asynchronous materialized views, replay, time bucketing, tenant isolation, cardinality control, hot-partition mitigation, and tiered retention.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 40 — Design a Distributed Job Scheduler
