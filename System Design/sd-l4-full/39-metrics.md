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


## Detailed reference design

### Understand the data path

Metrics/log platforms are usually **write-dominated** and append-oriented. Do not put an expensive query index in the critical ingestion path.

```text
Agents
  ↓ batch/compress
Ingest endpoints
  ↓
Durable stream/log
  ├→ raw/object storage
  ├→ realtime aggregators
  └→ indexers/search
```

Once the durable stream accepts data, downstream query systems can recover/replay independently.

### Batching

Sending one HTTP request per log line wastes overhead. Agents batch hundreds/thousands of events, compress, and retry with sequence/batch IDs.

Trade-off: bigger batches improve throughput/compression but increase delay and retry payload.

### Partitioning

Possible key:

```text
hash(tenant_id, metric/log stream)
```

Need to preserve enough locality while avoiding one giant tenant on one partition. Large tenants can receive dedicated partition sets.

For metrics aggregation, partitioning by metric+tenant may keep related series together. For raw logs, broader hash distribution may maximize ingest throughput.

### Raw vs derived storage

**Raw immutable events** in cheap object storage support replay/audit.

**Time-series aggregates** support fast charts:

```text
(tenant, metric, dimensions, time_bucket)
→ count/sum/min/max/percentile sketch
```

**Search index** supports text/filter log queries for recent retention.

Do not force one datastore to do all three jobs.

### Cardinality

Metrics labels/tags can create enormous unique time series:

```text
endpoint × user_id × request_id
```

High-cardinality dimensions explode memory/index size. Enforce allowed labels, quotas, or sample/aggregate. This is a major real-world production concern.

### Retention tiers

Example:

- recent 24h raw logs in fast searchable index;
- 30 days compressed object storage;
- metrics 1-minute resolution for 7 days;
- hourly rollups for 1 year.

Retention is product/cost driven.

### Query architecture

Query service chooses appropriate store based on time range/resolution. A dashboard for last 15 minutes reads fine-grained TSDB; one-year report reads rollups. This prevents scanning raw events for every graph.

### Backpressure

If downstream indexer lags, durable stream absorbs temporarily. If stream storage or ingest exceeds safe capacity, apply per-tenant quotas/load shedding. Never let one noisy customer cause all tenants to lose telemetry.

## Failure walkthrough

### Search cluster fails

Ingestion continues into durable stream/raw storage. Search queries degrade. When index returns, consumer replays missing events.

### Aggregator bug

Deploy corrected consumer and replay raw/stream history into a new versioned aggregate table, then cut over queries.

### One tenant 50× traffic

Quota/rate-limit, dedicate partitions, and isolate worker resources. Alert on tenant contribution and lag.

### Duplicate batches after retry

Use agent/batch IDs or tolerate duplicates in logs; metrics aggregation may dedupe by event ID if exactness matters or accept approximate duplicates if product allows.

## Interviewer follow-ups

### “Kafka or DB first?”

A durable stream decouples ingestion from query/index systems and supports replay at high write volume. At small scale, direct DB batching may be simpler. I introduce stream when throughput, burst isolation, or replay justify it.

### “How do you query arbitrary logs?”

Index selected recent fields/text in a search engine. Keep raw logs in object storage for long retention/reprocessing. Full indexing of all old data is expensive.

### “How do you compute percentiles?”

At interview depth, use streaming sketches/histograms rather than storing every latency in each aggregate. Exact algorithm can be abstracted unless asked.

## Common interview mistakes

- Search engine is ingestion source of truth.
- One request per event.
- No durable replay layer.
- No noisy-tenant isolation.
- High-cardinality tags ignored.
- Raw data retained forever in expensive hot storage.
- Query scans raw events for every dashboard.

## Short revision note

**Telemetry pattern:** batch ingest → durable stream → raw archive + aggregation + search derived views → tiered retention → tenant isolation/backpressure.

## Topics to revise

- [ ] batching/compression
- [ ] durable stream
- [ ] partition key
- [ ] raw vs derived
- [ ] time buckets/rollups
- [ ] high cardinality
- [ ] retention tiers
- [ ] replay/rebuild
- [ ] noisy-neighbor control

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll protect ingestion first: agents batch/compress into a durable stream, then independent consumers archive raw data, build time-series aggregates, and index recent logs. Query systems are derived and can fail/rebuild without losing accepted telemetry. Tenant quotas and cardinality control are first-class.

## How the design evolves at 10×

At 10× writes, increase partitions/ingest nodes and isolate giant tenants. At 10× retention, move older data to cheap object storage/rollups. At 10× queries, preaggregate and tier query stores rather than scanning raw events.

## Quick revision flashcards

**Why durable stream?**  
Decouples high-write ingestion from query/index health and enables replay.

**High cardinality?**  
Too many unique tag combinations explode series/index memory.

**Search down?**  
Ingest continues; replay index later.

**Noisy tenant?**  
Quota/dedicated partitions/resource isolation.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 40 — Design a Distributed Job Scheduler
