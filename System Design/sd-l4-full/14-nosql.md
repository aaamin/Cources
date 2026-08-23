# Lesson 14 — NoSQL Databases

**Phase:** Fundamentals  
**Session:** 14/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn the major NoSQL families—key-value, document, wide-column—their natural workloads, and how to compare them with relational storage without simplistic rules.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Key-value

A key-value store maps a key directly to a value, such as `session:abc → blob`. It is excellent for simple high-scale key lookups, but complex joins and ad hoc queries are limited.

## Document

Document stores keep JSON-like structured aggregates. They are convenient when a domain object is naturally read/written as a whole and schemas evolve. Embedding too much can create large updates and duplication.

## Wide-column

Wide-column systems are built for large distributed workloads with predictable partition-key access and sorted rows. They often require query-driven schema design and avoid arbitrary joins.

## Horizontal distribution

Many NoSQL systems make partitioning/replication first-class. They often trade relational flexibility for distribution simplicity. But modern SQL can also scale; the decision is about workload and guarantees, not slogans.

## Denormalization

Limited joins encourage duplicated, query-shaped data. This speeds reads but creates propagation, consistency, and repair complexity.

## Polyglot persistence

A large system may use SQL for orders, key-value for sessions, search for text, and object storage for media. Every new store adds operational burden and synchronization, so use it only when the capability is worth it.

## Other storage families you should recognize

NoSQL preparation should not stop at key-value/document/wide-column. At L4, you should also recognize when a specialized store is the natural fit.

### Search / inverted-index store

Search engines build structures such as inverted indexes that map terms to documents. They are useful for full-text search, filtering, ranking, and autocomplete-like retrieval. They are usually **derived indexes**, not the authoritative source of truth for payments or inventory.

### Time-series store

Time-series systems optimize timestamped measurements, retention, downsampling, and range queries such as:

```text
metric + dimensions + time range
```

They are natural for metrics, IoT telemetry, and monitoring data.

### Graph database

Graph databases model nodes and edges directly and can make relationship traversals natural: friends-of-friends, fraud networks, dependency graphs, or recommendation relationships.

Do not choose a graph database merely because the domain contains relationships. If the important queries are simple keyed lookups or bounded joins, a relational or key-value model may be simpler.

### Object storage

Object storage is optimized for large durable blobs rather than relational queries. Keep queryable metadata in a database and large bytes in object storage. Lesson 25 covers this in depth.

> **Important:** Specialized stores improve one access pattern while increasing operational complexity. Introduce them only when that access pattern is important enough to justify another system.

## Worked example — three workload choices

Session lookup by token naturally fits key-value. Orders/payments with strong relational invariants naturally fit SQL. Huge telemetry queried by device/time may fit wide-column or time-series storage. The correct answer changes with access patterns and guarantees.

## Interview lens

Never say “NoSQL scales better” without details. State the specific query pattern, transaction need, partitioning requirement, and operational cost.

## What to remember

NoSQL is a family of models, not one database category. Learn each model's natural access pattern and limitations.

## Review after reading

1. Good key-value workload?
2. Why documents?
3. Why query-driven wide-column schema?
4. Cost of denormalization?
5. Why is polyglot persistence expensive?

## Deeper study notes

### Start from capabilities, not brands

The useful interview question is not “MongoDB or Cassandra?” It is “Do I need key lookup, flexible document retrieval, range scans inside a partition, joins, transactions, full-text search, or global secondary queries?” Products change; workload capabilities are durable knowledge.

### Partition key is part of the schema

In many distributed NoSQL systems, the partition key determines where data lives and which queries are cheap. A schema can look logically correct but be physically unusable if the dominant query does not include the partition key. This is why query-driven modeling is emphasized so strongly.

### Consistency choices vary by store

Some NoSQL systems offer tunable read/write consistency; others expose transactions on limited scopes; some are eventually consistent by default. Do not infer consistency solely from the word “NoSQL.” State the guarantee your operation needs, then choose/configure a store that can provide it.

### Search is not “just another NoSQL database”

Search engines maintain inverted indexes optimized for text/filter/ranking. They are usually derived views rather than authoritative transaction stores. Keep source-of-truth data in a system suited to business correctness and update the search index asynchronously.

### Common mistakes

- Treating all NoSQL systems as equivalent.
- Assuming flexible schema means “no schema.” Applications still rely on field meaning and versioning.
- Using a key-value store for complex ad hoc reporting.
- Adding several storage technologies before one database becomes inadequate.
- Forgetting operational cost: backup, monitoring, compaction, partition repair, schema migration, and developer expertise.


## Important interview ideas

> **Important:** “NoSQL” is a family of different data models, not one database type. Always name the capability: key-value lookup, document aggregate, wide-column partitioned access, search index, etc.

### Key-value design

Natural API:

```text
get(key)
put(key, value)
delete(key)
```

This model is extremely powerful when the application knows the key. Sessions, rate-limit state, feature/config values, and simple profiles often fit well.

The difficulty begins with secondary access: “find every user in city X sorted by signup time” needs another index/model.

### Document design

A document can keep related fields together:

```json
{
  "user_id": 123,
  "name": "A",
  "preferences": {...}
}
```

This reduces joins for aggregate reads but can create large documents, duplicated nested data, and update contention if many independent fields change.

### Wide-column/query-shaped design

A distributed wide-column table often begins with a partition key and clustering/order columns. You design the table around a known query:

```text
partition: conversation_id
order: message_time/message_id
```

This can scale huge append workloads but is less flexible for arbitrary queries.

### “Schema-less” is misleading

Even if the database does not enforce a rigid schema, the application still depends on field names/types and needs version migration. Flexible schema reduces some migration friction; it does not eliminate data contracts.

### NoSQL and consistency

Do not assume every NoSQL system is eventual. Some support strong reads, conditional writes, transactions within partitions, or tunable quorum settings. Choose guarantees explicitly.

## Storage-family comparison

| Need | Natural starting point |
|---|---|
| relational transaction/joins | SQL |
| simple key lookup at huge scale | key-value |
| aggregate JSON documents | document |
| predictable partition+range query | wide-column |
| full-text/ranking | search index |
| huge blobs | object storage |

Often the source of truth and specialized derived stores coexist.

## Interview questions and model answers

### Q1. “Why choose NoSQL?”

Because the workload maps naturally to its data model and distribution characteristics—for example, key-based access with enormous horizontal throughput and no joins. “Because it scales” is insufficient because SQL systems also scale.

### Q2. “What is the cost of denormalization?”

More write/update paths, possible stale copies, extra storage, and repair logic. You gain fast predictable reads by accepting synchronization complexity.

### Q3. “Can a search engine be my main DB?”

Usually I treat search as a derived index because its strengths are text/filter/ranking rather than transactional business integrity. The authoritative record lives in a database suited to correctness; updates flow asynchronously to search.

### Q4. “When is one database better than polyglot persistence?”

Until a second datastore solves a clear important problem. Every additional store requires monitoring, backup, migration, security, consistency, and developer expertise. Simplicity is valuable.

## Common mistakes to avoid

- All NoSQL grouped together.
- “Schema-less means no schema.”
- “NoSQL is eventually consistent.”
- Using key-value for ad hoc reporting.
- Storing authoritative payments in a search index.
- Adding five databases to an L4 design without need.

## Short revision note

Choose the **data model that matches access patterns and guarantees**. NoSQL is a tool family, not a universal scalability answer.

## Topics to revise

- [ ] key-value
- [ ] document
- [ ] wide-column
- [ ] denormalization
- [ ] schema flexibility
- [ ] consistency options
- [ ] search as derived index
- [ ] polyglot persistence cost
- [ ] search/inverted-index stores
- [ ] time-series stores
- [ ] graph databases
- [ ] object storage as specialized store

## Interview-ready synthesis

### A strong 60–90 second explanation

I choose among key-value, document, wide-column, relational, search, and object storage by access pattern and guarantee. I avoid treating NoSQL as one capability or as a synonym for scale. If I introduce multiple stores, each must solve an important specialized workload that justifies its operational cost.

### How this topic connects to the wider system

- Performance: query-shaped NoSQL models can provide predictable high-throughput access.
- Correctness: transaction/consistency support varies; specify required guarantee instead of assuming.
- Scalability: partition-oriented stores are often designed for horizontal distribution.
- Operations: polyglot persistence multiplies backup, monitoring, security, and migration work.

### Revision flashcards with answers

**Key-value best for?**  
Direct lookup/update by known key.

**Document best for?**  
Aggregate-shaped records with flexible nested fields and limited joins.

**Wide-column best for?**  
Large distributed datasets with predictable partition + ordered-range queries.

**Schema-less?**  
Database may not enforce rigid schema, but applications still depend on data contracts/versioning.

**Search store source of truth?**  
Usually no; it is a derived text/filter/ranking index.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Rate limiter: key-value state maps naturally from policy key to counter/token state.
- Chat: wide-column/query-shaped storage can map conversation partition to ordered messages at extreme scale.
- Product search: search index is a derived specialized store while product truth may remain relational/document.

### When not to overuse this idea

Do not add a NoSQL store simply because the prompt mentions millions of users. First ask whether one SQL database with indexes/replicas still meets the workload.

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

**Next:** Lesson 15 — Database Replication & Failover
