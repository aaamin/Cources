# Session 14 — NoSQL & Specialized Datastores

## Outcome

You should be able to recognize the major NoSQL families, explain their access patterns and trade-offs, and know when a search engine, time-series database, graph database, or geospatial index is useful. You should choose storage from workload and correctness needs rather than popularity.

## The Core Principle

There is no universal “best database.”

Choose by asking:

```text
What are the reads?
What are the writes?
What must be strongly consistent?
What needs transactions?
What is the key?
What is the query shape?
How large and skewed is the dataset?
What needs ordering/range scans?
What operational complexity is acceptable?
```

## Key-Value Stores

Mental model:

```text
key → value
```

Example:

```text
"user:123" → {name, preferences, ...}
```

Strengths:
- fast point lookup by key;
- simple partitioning;
- easy cache/session/config use cases;
- often high scale.

Weaknesses:
- limited ad hoc querying;
- relationships/joins are not natural;
- secondary access paths may require additional structures.

Good use cases:
- session store;
- cache;
- feature/config lookup;
- shopping cart;
- simple entity lookup;
- rate-limit counters.

Do not say “key-value is O(1)” as a universal guarantee. Distributed/network/storage behavior still matters.

## Document Databases

Store document-shaped records, often JSON-like.

Example:

```json
{
  "orderId": "o1",
  "userId": "u1",
  "items": [
    {"productId": "p1", "qty": 2}
  ]
}
```

Strengths:
- flexible schema;
- natural aggregate/document storage;
- nested data;
- good when related data is read together.

Weaknesses:
- cross-document transactions/joins may be less natural or more expensive depending on database;
- large unbounded arrays/documents can be problematic;
- duplicating embedded data creates update complexity.

Good use:
- product/catalog content;
- user profile/preferences;
- content records with varying fields.

Use a document because the access pattern fits an aggregate—not because “schema can change.”

## Wide-Column Databases

Conceptually designed for massive partitioned datasets with access centered on a partition key and ordered clustering dimensions.

A simplified mental model:

```text
Partition key → many ordered rows/columns
```

Good for:
- huge write volume;
- time-ordered events;
- message/history storage;
- telemetry;
- workloads with known query patterns.

Strengths:
- horizontal partitioning;
- high write throughput;
- predictable key/range access.

Trade-offs:
- data model is very query-specific;
- joins/ad hoc queries are limited;
- bad partition key can create hot partitions;
- denormalization is common.

Example chat key:

```text
partition = conversation_id
cluster/order = message_sequence
```

But one giant conversation can become hot, so perhaps bucket by time/sequence.

## Search Engines / Inverted Indexes

Normal database indexes are not ideal for rich full-text relevance.

Conceptual inverted index:

```text
"pizza" → doc 1, doc 4, doc 9
"dhaka" → doc 4, doc 8
```

Search systems support:
- tokenization;
- full-text query;
- ranking;
- typo/fuzzy matching;
- faceting/filtering;
- sometimes geospatial search.

Common architecture:

```text
Primary DB = source of truth
     ↓
CDC/event/indexing pipeline
     ↓
Search index = derived query model
```

This means search may be slightly stale.

Do not put payment truth in Elasticsearch just because search is fast.

## Time-Series Databases

Optimized for timestamped measurements:

```text
(metric, labels/tags, timestamp, value)
```

Examples:
- CPU metrics;
- IoT sensors;
- application latency;
- financial ticks (depending on semantics).

Useful features conceptually:
- time partitioning;
- retention;
- downsampling;
- time-window aggregation;
- compression.

The hard part is often:
- high ingest;
- cardinality of labels;
- retention;
- aggregate query cost.

## Graph Databases

Model nodes/edges and graph traversal.

Example:

```text
User ─FOLLOWS→ User
User ─MEMBER_OF→ Group
```

Useful for:
- multi-hop relationships;
- fraud graphs;
- recommendation/exploration;
- dependency graphs.

But many “graph-like” products do not need a graph DB. A social follow table in SQL or adjacency lists in KV/wide-column may be sufficient if queries are simple one-hop lookups.

Use graph DB when traversal/query semantics justify it.

## Geospatial Storage / Indexing

Queries:

```text
drivers near this coordinate
restaurants within 5 km
places inside region
```

Options conceptually:
- geospatial DB index;
- search engine geo index;
- geohash/grid;
- quadtree;
- other spatial structures.

The important L4 point:
**latitude/longitude columns alone do not solve efficient nearby search at scale.**

## SQL vs NoSQL

Avoid false binary thinking.

Relational DB strengths:
- transactions;
- constraints;
- joins;
- flexible query;
- mature tooling.

NoSQL strengths vary by family:
- scale-out key access;
- flexible documents;
- massive partitioned writes;
- specialized query shapes.

A system can use both:
```text
Orders → relational DB
Product search → search index
Sessions → key-value cache
Metrics → time-series store
Images → object storage
```

But every extra technology increases operational complexity. Add only when justified.

## Denormalization in NoSQL

Since joins may be limited, duplicate data to match reads.

Example:

```text
PostsByAuthor(author_id, time, post_data)
PostsByHashtag(hashtag, time, post_data)
```

Same source post may appear in multiple projections.

Need:
- update strategy;
- eventual consistency expectations;
- rebuild/reconciliation.

## Partition Key Matters More Than Brand

A wide-column or key-value store with a terrible partition key still fails.

Bad:
```text
partition_key = current_date
```
for all writes today.

All writes hit one partition.

Better:
```text
tenant_id + bucket
device_id
conversation_id + bucket
```
depending on access pattern.

## Worked Example — Product Catalog

Requirements:
- product truth;
- category/filter queries;
- keyword search;
- many reads;
- moderate writes.

Possible:
```text
Relational/Document DB = product source
        ↓ changes
Search Index = text/filter query model
        ↓
Cache hot product pages
```

Why not search index only?
- source-of-truth updates/transactions/admin workflows may fit primary DB better;
- search indexing can lag.

Why not SQL only?
- SQL may handle simple filters fine; use search engine only if relevance/fuzzy/high-scale search requirements justify it.

## Comparison Table

| Store | Natural access | Strength | Common trap |
|---|---|---|---|
| Relational | keys, joins, ranges | constraints + transactions | assuming it cannot scale |
| Key-value | key → value | simple fast lookup | poor secondary queries |
| Document | aggregate/document | flexible nested data | huge/unbounded documents |
| Wide-column | partition + ordered range | large-scale writes | hot/bad partition key |
| Search | text/filter/rank | relevance + inverted index | treating derived index as truth |
| Time-series | time-window metrics | retention/aggregation | tag/cardinality explosion |
| Graph | multi-hop traversal | relationship queries | using it for simple one-hop data |

## Small Design Drills

1. Why might chat history fit a wide-column model?
2. Why is product search often a derived index rather than primary truth?
3. Does flexible schema eliminate the need for data modeling?
4. When is graph DB unnecessary for a social network?
5. What makes a partition key bad?
6. Why is adding five database technologies costly?

<details>
<summary>Answer key</summary>

1. Known partition by conversation and ordered message range access can fit well.
2. Search systems optimize query/relevance and may lag; durable transactional product source often belongs elsewhere.
3. No. Access patterns, ownership, validation, migration, and query shape still matter.
4. If only simple one-hop follower queries are needed, relational/KV/adjacency structures may suffice.
5. It creates skew/hot partitions, poor locality, or requires scatter-gather for common reads.
6. More operations, backups, monitoring, expertise, failure modes, and consistency pipelines.

</details>

## Common Interview Mistakes

- “NoSQL scales, SQL does not.”
- Choosing document DB only for schema flexibility.
- Using graph DB because data has relationships.
- Making search index authoritative without discussing consistency/durability.
- Ignoring partition key.
- Adding specialized stores for small/simple workloads.
- Forgetting eventual consistency of derived indexes.
- Treating all NoSQL systems as the same.

## Must Remember

- **NoSQL is a family, not one model.**
- **Key-value is optimized around key lookup.**
- **Document stores fit aggregate-shaped data.**
- **Wide-column designs depend heavily on partition/access keys.**
- **Search indexes are often derived from a primary source.**
- **Time-series stores optimize timestamped ingest/retention/query.**
- **Graph DBs are valuable for real traversal workloads, not every relationship.**
- **Choose storage from access patterns and invariants.**
- **Every extra datastore adds operational cost.**

## Interview Revision Summary

Storage choice checklist:

```text
Transactions/invariants?
Point lookup?
Range/order?
Document aggregate?
High write partitioned history?
Full-text relevance?
Time-window aggregation?
Multi-hop graph traversal?
Geo proximity?
Primary or derived data?
Consistency?
Operational complexity?
```

## Explain Without Notes

Design storage choices for an e-commerce system with transactional orders, product search, session carts, images, and analytics metrics. Justify why each store exists.

## Completion Checklist

- [ ] I can distinguish NoSQL families.
- [ ] I recognize specialized store use cases.
- [ ] I do not equate NoSQL with “better scale.”
- [ ] I understand primary vs derived storage.
- [ ] I choose based on access patterns/consistency.
