# Session 12 — Database Indexing & Query Access Paths

## Outcome

You should be able to inspect a query/access pattern, choose useful indexes, explain composite-index ordering/selectivity, understand covering indexes and write/storage costs, and know when full-text/geospatial/external search indexes are appropriate.

## Why Indexes Exist

Without an index, a database may scan many rows to find a small result.

```text
1,000,000 rows
query by email
```

A suitable index lets the database navigate directly to matching rows rather than examine all rows.

Indexes trade:
- additional storage;
- additional write work;
- maintenance complexity

for faster reads.

## Primary Index / Primary Key

Primary key uniquely identifies rows.

Example:

```sql
SELECT * FROM users WHERE id = ?
```

Databases commonly organize/index the primary key efficiently.

## Secondary Index

Index on another access path.

Example:
```sql
CREATE INDEX idx_users_email ON users(email);
```

Supports:
```sql
SELECT * FROM users WHERE email = ?;
```

If email must be unique, prefer a unique index/constraint.

## Composite Index

Index multiple columns:

```sql
CREATE INDEX idx_orders_user_created
ON orders(user_id, created_at);
```

Good for:
```sql
WHERE user_id = ?
ORDER BY created_at DESC
```

### Column order matters

Conceptually, `(user_id, created_at)` is organized first by `user_id`, then by `created_at` within that user.

It may support:
```text
user_id = X
user_id = X AND created_at > ...
```

It is generally less useful for:
```text
created_at = ...
```
without filtering by the leading column, though exact optimizer behavior varies.

Design from actual queries.

## Selectivity

An index is most useful when it narrows data significantly.

High selectivity:
- email;
- order ID.

Low selectivity:
- boolean `is_active` where 95% are true.

A low-selectivity index may still help in combination with other columns or if the minority value is queried, but do not assume every filter deserves its own index.

## Covering Index Concept

If an index contains all columns needed for a query, the DB may satisfy it without visiting the base row/table page.

Example query:
```sql
SELECT created_at, total
FROM orders
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```

An index including `user_id`, `created_at`, and perhaps `total` can reduce extra reads.

Trade-off: larger index, more write cost.

## Sorting and Pagination

Index order can avoid expensive sort operations.

For cursor pagination:

```sql
WHERE user_id = ?
  AND created_at < :cursor
ORDER BY created_at DESC
LIMIT 20
```

Index:
```text
(user_id, created_at)
```

This is usually more scalable than large offset scans.

## Write Amplification

Every insert/update/delete may also modify indexes.

If a table has 12 indexes:
- writes touch many structures;
- storage grows;
- cache pressure increases.

Do not index every column “just in case.”

## Read Amplification

A poor query might require:
- index lookup;
- many random base-row reads;
- multiple indexes;
- large range scan.

Indexes reduce some read work but can still be inefficient if a query returns huge portions of the table.

## Query-Driven Indexing Process

Use this repeatable method:

### 1. Write access pattern

```text
List latest 50 orders for a user
```

### 2. Express query

```sql
SELECT ...
FROM orders
WHERE user_id=?
ORDER BY created_at DESC
LIMIT 50;
```

### 3. Identify equality/range/order columns

- equality: user_id
- order/range: created_at

### 4. Propose index

```text
(user_id, created_at DESC)
```

### 5. Consider returned columns and write cost

Do not begin from “what indexes should this table have?” Begin from reads/writes.

## Full-Text Search

Normal B-tree-style indexes are not enough for rich text search such as:

```text
"restaurants serving spicy ramen near me"
```

Search engines/full-text indexes use tokenization/inverted-index concepts.

At recognition level:

```text
term → documents containing term
```

Use when requirements include:
- keyword relevance;
- stemming/tokenization;
- typo tolerance;
- ranking;
- complex text search.

Often:
```text
Primary DB = source of truth
Search index = derived query system
```

Updates propagate asynchronously, so search freshness may be eventually consistent.

## Geospatial Index

Needed for queries such as:

```text
drivers within 2 km
restaurants near coordinate
```

Databases/search systems can index spatial data.

At interview depth, know:
- geographic search needs a spatial indexing strategy;
- geohash/grid/quadtree are alternative conceptual approaches;
- regular latitude range scans can become inefficient.

## Secondary Index Across Shards

When data is sharded by `user_id`, querying by `email` may not know which shard owns the user.

Options:
- global directory/index;
- duplicate lookup mapping;
- scatter-gather;
- choose a different partition key if that access dominates.

Sharding complicates secondary access paths.

## Worked Example — Orders

Access patterns:

1. Get order by ID.
2. List latest orders for a user.
3. Admin list pending orders by creation time.
4. Search by external payment ID.

Possible indexes:

```text
PRIMARY KEY(order_id)
INDEX(user_id, created_at DESC)
INDEX(status, created_at)
UNIQUE(external_payment_id)
```

Do not add:
```text
INDEX(total)
INDEX(currency)
INDEX(is_discounted)
```
unless actual queries justify them.

## Small Design Drills

1. Query: `WHERE tenant_id=? AND created_at>? ORDER BY created_at`. Suggest a composite index.
2. Why can too many indexes reduce write throughput?
3. Why might index on boolean `active` be weak?
4. Why is cursor pagination often better than offset=1,000,000?
5. What storage is commonly used for rich full-text relevance search?
6. A DB is sharded by user ID but lookup by email is frequent. What issue appears?

<details>
<summary>Answer key</summary>

1. `(tenant_id, created_at)` is a natural starting point.
2. Every write must maintain more index structures and storage.
3. Low selectivity; it may match most rows.
4. Cursor can seek from indexed position instead of skipping huge result sets.
5. Full-text/search engine/index (or DB full-text feature when sufficient).
6. Email does not identify the shard; need global mapping/index, scatter-gather, or reconsider partitioning.

</details>

## Common Interview Mistakes

- “Index every filter column.”
- Ignoring composite index order.
- Ignoring write amplification.
- Assuming an index makes a query O(1).
- Using offset pagination at huge depths.
- Treating search engine as source of truth without reason.
- Forgetting derived search-index freshness.
- Ignoring secondary access after sharding.

## Must Remember

- **Indexes exist to serve access patterns.**
- **Composite index order matters.**
- **High selectivity often improves usefulness.**
- **Indexes cost storage and write throughput.**
- **Cursor pagination works well with ordered composite indexes.**
- **Full-text relevance usually needs a specialized index/search system.**
- **Geospatial queries need spatial indexing.**
- **Sharding complicates secondary indexes.**

## Interview Revision Summary

For each read:
```text
Filter?
Equality columns?
Range?
Sort?
Limit?
Returned columns?
Selectivity?
Index?
Write cost?
Shard locality?
```

## Explain Without Notes

Design indexes for an order table supporting:
- order ID lookup;
- latest orders by user;
- pending orders by time;
- payment ID lookup.

## Completion Checklist

- [ ] I understand primary/secondary/composite indexes.
- [ ] I understand selectivity.
- [ ] I can explain covering indexes conceptually.
- [ ] I design indexes from queries.
- [ ] I understand write amplification.
- [ ] I recognize full-text/geospatial index needs.
