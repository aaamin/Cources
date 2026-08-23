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
