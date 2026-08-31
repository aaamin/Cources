# Session 13 — Data Modeling, Access Patterns & Read Models

## 1. Must Learn

### Access-pattern-first modeling
- **Understand:** List primary reads/writes before choosing schema or database.
- **Decision/trade-off:** Flexible generic model vs workload-optimized model.

### Entities, relationships & invariants
- **Understand:** Identify core entities, ownership, uniqueness, ordering, and strong-consistency needs.
- **Decision/trade-off:** Model simplicity vs preserving business correctness.

### Source of truth
- **Understand:** Make authoritative data ownership explicit.
- **Decision/trade-off:** Single authoritative ownership vs duplicated derived copies.

### Normalized write model
- **Understand:** Keep authoritative writes consistent and avoid unnecessary duplication.
- **Decision/trade-off:** Write correctness/maintainability vs read joins/latency.

### Denormalized read models
- **Understand:** Duplicate/precompute data to serve important reads efficiently.
- **Decision/trade-off:** Read speed vs staleness/synchronization complexity.

### Materialized/precomputed views & derived data
- **Understand:** Understand maintaining query-oriented derived representations.
- **Decision/trade-off:** Query efficiency vs update pipeline/rebuild complexity.

## 2. Should Know

- Separate data-model choice from database-product choice.
- Think about retention and ordering where they affect storage shape.
- Know that read models can be rebuilt only if the authoritative data/event source allows it.

## 3. Recognition Only

- CQRS at recognition depth here
- Materialized-view implementation details

## 4. Important Comparisons

- Write model vs read model.
- Normalized vs denormalized representation.
- Source-of-truth data vs derived data.
- Model-first vs database-product-first design.
- Compute on read vs precompute on write.

## 5. Important Interview Questions

1. What are the top reads and writes?
2. What must be unique or strongly consistent?
3. What needs ordering?
4. Which entity/service owns the source of truth?
5. What data is worth precomputing?
6. How is duplicated read-model data refreshed or rebuilt?

## 6. Common Interview Mistakes

- **Picking a database before access patterns** → Model requirements first, then choose storage.
- **One schema for every access pattern** → Use derived read models when high-value reads need them.
- **No source-of-truth ownership** → State which copy is authoritative.
- **Denormalizing without sync strategy** → Explain update/rebuild behavior.
- **Over-normalizing a read-heavy hot path** → Balance correctness with access cost.

## 7. Communication

### Important Vocabulary

access pattern, entity, source of truth, normalized write model, denormalized read model, materialized view, precomputed view, derived data, ownership, invariant

### Useful Interview Phrases

- “I’ll list the dominant access patterns before choosing the schema.”
- “This table is authoritative; this other representation is a derived read model.”
- “I’d denormalize this only because it removes a critical read-time cost.”

### Important Questions to Ask the Interviewer

- **Question:** “What are the most frequent/latency-sensitive reads?”  
  **Why it matters:** Determines read-model needs.
- **Question:** “Which fields must be unique or strongly consistent?”  
  **Why it matters:** Determines authoritative model and constraints.
- **Question:** “How fresh must derived data be?”  
  **Why it matters:** Determines sync/update strategy.

## 8. ⭐ Must Remember

1. Access patterns drive modeling.
2. Separate model choice from database-product choice.
3. Make source-of-truth ownership explicit.
4. Normalize authoritative writes when useful.
5. Denormalize selectively for important reads.
6. Every derived copy needs freshness/rebuild reasoning.

## 9. Study Priority

1. Study first: access patterns, entities, ownership/invariants.
2. Study next: normalized write model vs denormalized read model.
3. Finish with: precomputed views, derived data, freshness/rebuild.

## 10. Revision Checklist

- [ ] List reads/writes before modeling.
- [ ] Identify source of truth and invariants.
- [ ] Design a write model and optional read model.
- [ ] Explain why denormalization is justified.
- [ ] Explain how derived data stays usable.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
