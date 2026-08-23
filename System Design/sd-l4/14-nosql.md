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
