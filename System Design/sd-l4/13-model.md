# Lesson 13 — Data Modeling & Access Patterns

**Phase:** Fundamentals  
**Session:** 13/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn to derive schemas from requirements, access patterns, ordering, invariants, ownership, and retention before choosing a database product.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Entities and ownership

Identify durable domain concepts—User, Order, Message, File, Reservation—and decide which service/data store owns each truth. Ambiguous shared ownership creates synchronization and correctness problems.

## Access patterns first

List critical reads and writes before schema design. For social posts: create post, get by ID, list user's recent posts, get follow graph, build feed. Each path implies keys, indexes, and possibly derived data.

## Invariants

An invariant must remain true: username unique, seat not double-sold, confirmed order has successful payment. Invariants determine transaction boundaries and consistency requirements.

## Ordering and pagination

Feeds, messages, and logs need stable ordering. Store an ordering field plus tie-breaker where needed. Cursor pagination based on stable keys is usually better than large offsets for constantly changing datasets.

## Denormalization

One normalized model may not serve every high-scale read efficiently. Derived tables or duplicated fields can optimize hot access patterns, but they introduce staleness, update fan-out, and repair work.

## Retention

Temporary and historical data have different lifecycles. Retention affects partition layout, archive tier, backup cost, privacy deletion, and cleanup jobs.

## Worked example — social-post model

Start with User, Post, Follow. Reads include `post by ID` and `recent posts by author`. A table keyed only by `post_id` does not efficiently serve author history, so add `(author_id, created_at)` indexing or a derived user-post list. The access pattern, not abstract normalization alone, determines the useful shape.

## Interview lens

Say your access patterns aloud before naming storage. This naturally explains keys, indexes, partitioning, and consistency choices.

## What to remember

Good modeling connects **entities + ownership + access patterns + invariants + ordering + retention**.

## Review after reading

1. Why list access patterns first?
2. What is an invariant?
3. How does pagination affect indexing?
4. Cost of denormalization?
5. Why model retention?

## Deeper study notes

### Model state transitions, not just nouns

For correctness-heavy systems, the lifecycle matters as much as the entity fields. A ticket can move `AVAILABLE → HELD → SOLD` or back to `AVAILABLE` after expiry. An order can be `PENDING → PAID → FULFILLED` or `CANCELLED`. Explicit state machines expose invalid transitions and failure-recovery cases.

### Source of truth vs derived views

A feed table, search index, cache, analytics warehouse, and materialized aggregate may all contain copies of the same underlying facts. Label which store is authoritative. Derived views can be rebuilt and can often tolerate lag; the source of truth should preserve business invariants.

### Ownership reduces distributed transactions

If Order Service owns order state, other services should consume APIs/events rather than updating the order tables directly. Clear ownership reduces conflicting writers and makes schema changes safer.

### Design for the hardest query

A schema that serves `GET by ID` but not the product's dominant list/range query is incomplete. For each entity, write 3–5 highest-value access patterns, their expected cardinality, ordering, and consistency. That becomes the basis for indexes and partition keys.

### Common mistakes

- Modeling only entities, not state transitions.
- Using timestamps alone as globally unique cursors.
- Creating denormalized copies without a repair strategy.
- Ignoring deletion/retention in immutable event-heavy systems.


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

**Next:** Lesson 14 — NoSQL Databases
