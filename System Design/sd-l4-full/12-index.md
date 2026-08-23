# Lesson 12 — Database Indexing

**Phase:** Fundamentals  
**Session:** 12/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn why indexes accelerate reads, how secondary/composite/covering indexes work conceptually, and why every index adds write and storage cost.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Why indexes exist

Without an index, the database may scan many rows to find matches. An index is an additional lookup structure that narrows the search. It trades storage and maintenance work for faster reads.

## Primary and secondary indexes

The primary key provides the main identity path. Secondary indexes provide alternate lookup paths. If users are keyed by `user_id` but queried by email, an email index can avoid a full scan.

## Composite indexes

A composite index such as `(user_id, created_at)` supports queries constrained by `user_id` and ordered/ranged by `created_at`. Column order matters; the same index may be poor for a query using only `created_at`.

## Covering indexes

A covering index contains all fields needed by a query, allowing the DB to avoid fetching the base row. That can reduce I/O but makes the index larger and increases write amplification.

## Write amplification

Each insert/update/delete may update multiple indexes. Too many indexes slow writes, consume storage, and increase maintenance. “Index every column” is not a design strategy.

## Derive from access patterns

Start with the actual query. For `WHERE conversation_id=? ORDER BY created_at DESC LIMIT 50`, a composite index beginning with `conversation_id` and then `created_at` naturally follows.

## Worked example — message history index

A Message row has `message_id, conversation_id, sender_id, created_at, content`. The dominant read is “latest 50 messages in one conversation.” An index `(conversation_id, created_at)` directly serves that path. An index on `sender_id` does not.

## Interview lens

Tie every index to a real query and mention its write/storage cost. This shows you understand both benefit and trade-off.

## What to remember

Indexes are materialized lookup structures. The important chain is **access pattern → index → faster read / more write cost**.

## Review after reading

1. Why is a full scan expensive?
2. Secondary index?
3. Why does composite order matter?
4. Covering index?
5. Why can too many indexes hurt?

## Deeper study notes

### Think in selectivity

An index is most useful when it narrows the result dramatically. An index on a boolean `is_active` where 95% of rows are true may provide little benefit. An index on unique email is highly selective. The optimizer considers selectivity, table size, and cost when choosing a plan.

### Composite prefix rule intuition

For an index `(tenant_id, created_at, id)`, queries beginning with `tenant_id` can efficiently locate a tenant's range. A query on only `created_at` cannot generally jump to one contiguous part of the index because timestamps are interleaved under each tenant. This is why column order follows access patterns.

### Sorting matters

Indexes can sometimes satisfy both filtering and ordering. A timeline query filtered by user and sorted by time is a classic example. If the DB can walk the index in the desired order and stop after 50 rows, it avoids sorting a large result set.

### Indexes affect storage design

In distributed databases, a “secondary index” may require scatter-gather or a separately maintained global index. Do not assume every index has the same cost as a single-node B-tree. Ask whether the index is local to a partition or globally distributed.

### Common mistakes

- Adding an index without naming the query it serves.
- Ignoring write amplification.
- Building redundant indexes such as `(a)` and `(a,b)` without checking whether both are needed.
- Assuming an index always makes a query faster; small tables/full scans can be cheaper.


## Important interview ideas

> **Important:** An index is a data structure maintained to make specific access patterns fast. Every index is a trade: faster selected reads, slower/more expensive writes and extra storage.

### Think in query shapes

Do not say “index `created_at`.” Start with the query:

```sql
SELECT message_id, sender_id, body, created_at
FROM messages
WHERE conversation_id = ?
  AND created_at < ?
ORDER BY created_at DESC
LIMIT 50;
```

A composite index beginning with `(conversation_id, created_at)` matches this shape. The leading column narrows to one conversation; the second supports ordered range navigation.

### Composite-index prefix rule intuition

For an index `(A, B, C)`, queries filtering by `A`, `(A,B)`, or `(A,B,C)` can commonly use the ordered structure effectively. A query filtering only `B` usually cannot efficiently jump to all B values because the index is first organized by A.

Exact optimizer behavior differs by database, but this mental model is interview-useful.

### Index selectivity

An index on a column with two values, such as `is_active`, may not help much if half the table matches. A highly selective email or user ID index is more useful. Composite indexes can combine a low-selectivity field with another condition if the actual query benefits.

### Write amplification and memory

Indexes consume disk and cache memory. Every insert/update may update several B-tree/LSM structures. A table with ten unnecessary indexes can have much lower write throughput.

This matters for write-heavy logs, telemetry, and feeds.

### Covering indexes

If an index contains all fields required by a hot query, the database may answer without fetching the base row. This can reduce random I/O. But larger indexes cost more memory and write work.

## Worked scenario — user order history

Query:

```sql
SELECT order_id, status, total, created_at
FROM orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

Candidate index:

```text
(user_id, created_at DESC)
```

If this is an extremely hot query and the DB supports included columns, adding `status,total` as included/covering data may avoid base-table lookups. You would not do this automatically; profile the read benefit vs increased index size/write cost.

## Interview questions and model answers

### Q1. “Why not index every field?”

Each index consumes storage and must be updated on writes. Too many indexes increase write latency, compaction/maintenance, memory use, and operational work. I index the important query patterns, not every possible future filter.

### Q2. “Primary vs secondary index?”

The primary key is the main row identity and often determines physical/clustered organization depending on the engine. A secondary index provides an additional lookup path such as email→row. Exact physical details vary; the conceptual distinction is sufficient for system design.

### Q3. “What is an index-only/covering query?”

The index contains every field needed to answer the query, so the engine may avoid fetching the base row. This improves some read paths but creates a wider, more expensive index.

### Q4. “Why can an index be bad for low-cardinality data?”

If most rows match, scanning the index plus fetching many rows may cost as much as or more than a sequential scan. Index usefulness depends on selectivity, query shape, ordering, and data size.

## Common mistakes to avoid

- Listing indexes without corresponding queries.
- Wrong composite column order.
- Indexing every column.
- Ignoring write-heavy cost.
- Assuming an index makes every query O(1).
- Forgetting pagination/order requirements.

## Short revision note

**Query → filter/order → index columns/order → read benefit → write/storage cost.** That is the interview reasoning chain.

## Topics to revise

- [ ] primary vs secondary index
- [ ] composite indexes
- [ ] leading-column intuition
- [ ] selectivity
- [ ] covering index
- [ ] read/write amplification
- [ ] indexes for pagination
- [ ] index trade-offs

## Interview-ready synthesis

### A strong 60–90 second explanation

I derive indexes from query shapes. I look at equality filters, range/order, pagination, and selected fields, then choose a primary/secondary/composite index and explain write/storage cost. Composite column order matters because the structure is ordered by its leading fields.

### How this topic connects to the wider system

- Performance: good indexes avoid full scans and can support ordered pagination.
- Cost: every index consumes disk/cache memory and increases write work.
- Data modeling: access patterns determine index design.
- Scalability: an index can postpone sharding by making a single database far more efficient.

### Revision flashcards with answers

**Composite index?**  
One index ordered by multiple columns, useful when query filters/orders align with the prefix.

**Selectivity?**  
How narrowly a condition filters rows; highly selective fields often benefit more from indexes.

**Covering index?**  
Contains all columns needed for a query so base-row fetch may be avoided.

**Why index order matters?**  
The structure is first organized by leading columns; skipping them may prevent efficient seek.

**Why not index every field?**  
Storage and write amplification increase, and many indexes provide little value.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Chat history: `(conversation_id, created_at/id)` supports ordered pagination.
- Order history: `(user_id, created_at)` supports recent orders without full scans.
- Scheduler: index on `next_run_at` is the difference between finding due jobs efficiently and scanning millions of future jobs.

### When not to overuse this idea

Do not add indexes for speculative queries with no importance. A write-heavy system can spend more time maintaining indexes than serving useful reads.

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

**Next:** Lesson 13 — Data Modeling & Access Patterns
