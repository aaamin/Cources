# Session 14 — NoSQL & Specialized Datastores

## 1. Must Learn

### Key-value stores
- **Understand:** Understand key-based lookup model and why it fits simple high-scale access patterns.
- **Decision/trade-off:** Simple scalable access vs limited query flexibility.

### Document databases
- **Understand:** Understand document-oriented records and flexible schema.
- **Decision/trade-off:** Developer/schema flexibility vs joins/relational constraints/query predictability.

### Wide-column databases
- **Understand:** Understand partition-key-oriented storage for massive distributed workloads conceptually.
- **Decision/trade-off:** High scalable patterned access vs restrictive data modeling/querying.

### NoSQL design traits
- **Understand:** Know horizontal scaling, denormalization, flexible schemas, and query limitations are workload-dependent—not universal benefits.
- **Decision/trade-off:** Scale/flexibility vs stronger relational features.

### Storage selection by access pattern
- **Understand:** Choose store family from reads/writes, consistency, query needs, scale, operations, and cost.
- **Decision/trade-off:** Specialization vs operational complexity.

### Primary store vs specialized secondary system
- **Understand:** Know search/time-series/graph/geo systems may complement rather than replace the primary database.
- **Decision/trade-off:** Best tool for query vs duplicate data and synchronization.

## 2. Should Know

- Search engine + inverted index conceptually.
- Time-series, graph, and geospatial stores: know the workload each targets.
- Recognize that NoSQL systems differ greatly in consistency and transaction support.

## 3. Recognition Only

- Vector databases
- Storage-engine internals
- Vendor-specific partition implementations

## 4. Important Comparisons

- SQL vs key-value vs document vs wide-column.
- Primary database vs secondary search/index system.
- Flexible schema vs enforced relational schema.
- General-purpose store vs specialized store.

## 5. Important Interview Questions

1. Why not use SQL here?
2. What exact access pattern does this NoSQL store make efficient?
3. What queries become difficult?
4. What consistency/invariant requirements could make this choice risky?
5. When is a search engine better as a secondary index instead of source of truth?
6. What operational complexity does adding another datastore create?

## 6. Common Interview Mistakes

- **“NoSQL scales, SQL doesn't”** → Discuss actual partitioning/access-pattern needs.
- **Choosing by popularity** → Tie store family to workload.
- **Assuming schema-less means no schema** → Application data still has structure/evolution concerns.
- **Using specialized store as source of truth automatically** → Separate authoritative storage from secondary query systems.
- **Ignoring query limitations** → State what becomes hard.

## 7. Communication

### Important Vocabulary

key-value, document database, wide-column, flexible schema, denormalization, access pattern, query limitation, inverted index, time-series, graph database, geospatial, secondary index

### Useful Interview Phrases

- “I’m choosing this store because the dominant access pattern is …”
- “The trade-off is scalable keyed access at the cost of richer relational querying.”
- “I’d keep the primary source of truth here and feed a specialized search index.”

### Important Questions to Ask the Interviewer

- **Question:** “What are the exact query patterns?”  
  **Why it matters:** Primary driver of store family.
- **Question:** “Do we need multi-record transactions or relational constraints?”  
  **Why it matters:** May favor SQL.
- **Question:** “Is specialized search/graph/time-series querying core or secondary?”  
  **Why it matters:** Determines primary vs auxiliary role.

## 8. ⭐ Must Remember

1. NoSQL is not one database model.
2. Choose storage from access patterns and correctness needs.
3. Horizontal scaling often comes with modeling/query trade-offs.
4. Denormalization is common but creates synchronization cost.
5. Specialized stores often complement a primary store.
6. Operational complexity matters.

## 9. Study Priority

1. Study first: key-value, document, wide-column and SQL comparison.
2. Study next: selection principles and query/consistency trade-offs.
3. Finish with: search/time-series/graph/geo recognition.

## 10. Revision Checklist

- [ ] Choose a store family for a workload and defend it.
- [ ] Explain what queries become harder.
- [ ] Discuss consistency and operational trade-offs.
- [ ] Distinguish primary store from secondary specialized system.
- [ ] Avoid generic “NoSQL scales better” reasoning.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
