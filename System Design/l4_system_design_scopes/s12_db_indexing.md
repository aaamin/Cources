# Session 12 — Database Indexing & Query Access Paths

## 1. Must Learn

### What an index does
- **Understand:** Understand an index as an additional access path that speeds selected reads at storage/write cost.
- **Decision/trade-off:** Read latency vs write amplification and storage.

### Primary vs secondary indexes
- **Understand:** Know primary/key-oriented access vs additional indexes on other fields.
- **Decision/trade-off:** Fast access flexibility vs maintenance cost.

### Composite indexes & ordering
- **Understand:** Understand multi-column indexes and why column order/prefix matters for supported queries.
- **Decision/trade-off:** One useful ordered index vs many overlapping indexes.

### Selectivity & query-driven indexing
- **Understand:** Choose indexes from actual filters/sorts/lookups; high-selectivity predicates often benefit more.
- **Decision/trade-off:** Index usefulness vs low-selectivity overhead.

### Covering index concept
- **Understand:** Know an index can contain enough fields to answer a query without extra row lookup.
- **Decision/trade-off:** Faster reads vs larger indexes/write cost.

### Read/write amplification & storage cost
- **Understand:** Every extra index can reduce read work while increasing write/storage work.
- **Decision/trade-off:** Read-heavy optimization vs write throughput/cost.

### Special access paths
- **Understand:** Know full-text/geospatial indexes conceptually and when an external search system may fit better.
- **Decision/trade-off:** Database simplicity vs specialized query capability/operational complexity.

## 2. Should Know

- Understand that indexes can help filtering, joining, and ordering depending on shape/order.
- Do not memorize database-specific EXPLAIN syntax; reason from access patterns.
- Too many indexes can become a write bottleneck.

## 3. Recognition Only

- B-tree vs LSM internals
- Bitmap indexes
- Index-only scan terminology

## 4. Important Comparisons

- Primary vs secondary index.
- Single-column vs composite index.
- Covered vs non-covered query.
- Database index vs external search index.
- More indexes vs write/storage cost.

## 5. Important Interview Questions

1. What are the dominant queries?
2. Which columns are used for equality filters, ranges, sorting, or joins?
3. Why does column order matter in a composite index?
4. When is an index too low-value to justify its cost?
5. Why can adding indexes slow writes?
6. When should search move to a specialized search index?

## 6. Common Interview Mistakes

- **Indexing every column** → Index query paths, not schema fields indiscriminately.
- **Ignoring composite order** → Match ordering to common query predicates/sorts.
- **Optimizing reads without write cost** → Call out index maintenance and storage.
- **Assuming an index always helps** → Selectivity/query shape determine usefulness.
- **Using DB indexes for advanced text search automatically** → Consider specialized search when query needs justify it.

## 7. Communication

### Important Vocabulary

primary index, secondary index, composite index, covering index, selectivity, access path, prefix, read amplification, write amplification, full-text index, geospatial index

### Useful Interview Phrases

- “I’d derive the index from this exact access pattern.”
- “The composite order matters because the query filters by … first.”
- “This improves reads but every write now has another index to maintain.”

### Important Questions to Ask the Interviewer

- **Question:** “What are the top read queries and sort orders?”  
  **Why it matters:** Drives index choice.
- **Question:** “How write-heavy is this table?”  
  **Why it matters:** Determines index budget.
- **Question:** “Do we need text relevance or geo queries beyond ordinary indexing?”  
  **Why it matters:** May justify a specialized store/index.

## 8. ⭐ Must Remember

1. Indexes are extra access paths, not free performance.
2. Design indexes from queries.
3. Composite column order matters.
4. Selectivity affects usefulness.
5. More indexes increase write amplification and storage.
6. Specialized search/geo needs may deserve specialized indexes.

## 9. Study Priority

1. Study first: index purpose, primary/secondary, query-driven indexing.
2. Study next: composite order, selectivity, covering indexes.
3. Finish with: amplification, storage cost, full-text/geo recognition.

## 10. Revision Checklist

- [ ] Given a query, propose an index.
- [ ] Explain why composite order works.
- [ ] Discuss read benefit vs write/storage cost.
- [ ] Recognize when an index will not help much.
- [ ] Know when external search may be more appropriate.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
