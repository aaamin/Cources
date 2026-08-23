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


## Important interview ideas

> **Important:** Data modeling is not drawing nouns. A strong model captures **access patterns, ownership, invariants, ordering, lifecycle, and state transitions**.

### Access-pattern table

Before schema design, write something like:

| Operation | Key/filter | Order | Consistency | Frequency |
|---|---|---|---|---|
| get post | post_id | none | read-after-write | high |
| list user posts | author_id | newest first | slight stale OK | high |
| create post | author_id | — | durable | medium |
| delete post | post_id | — | eventually remove derived copies | low |

This immediately tells you what indexes/partitioning and derived views may be necessary.

### State machines are part of data modeling

Many entities have lifecycle states:

```text
Order: PENDING → PAID → FULFILLED → CANCELED
Hold: ACTIVE → CONFIRMED / EXPIRED
Job: SCHEDULED → RUNNING → SUCCEEDED / FAILED
```

The allowed transitions and invariants are often more important than the columns. A system can have a beautiful schema and still be incorrect if invalid transitions are possible.

### Source of truth vs derived views

A feed table, search index, cache, analytics warehouse, and materialized aggregate may all copy the same facts. Label which is authoritative. Derived views can often be rebuilt and tolerate lag.

Example:

```text
Post DB = source of truth
Search index = derived
Feed cache = derived
Analytics warehouse = derived
```

Deleting a post means propagating/tombstoning it across those views.

### Ownership reduces distributed writes

If two services both directly update the same table, ownership is unclear and invariants become hard to enforce. Prefer one logical owner of business state. Other services interact via API/events.

## Worked scenario — messaging data model

Core entities:

```text
Conversation
Membership
Message
```

Access patterns:

- fetch conversation membership for authorization;
- append message;
- list messages by conversation in order;
- fetch conversations for one user;
- resume after last seen message.

This suggests a message key/order based on `(conversation_id, sequence/time)` and a separate user→conversation membership index/view. Trying to make one table serve every access pattern may be inefficient.

## Interview questions and model answers

### Q1. “What do you model first?”

I identify main entities and critical user operations, then write access patterns and invariants. Schema and database selection come after. For systems with lifecycle, I also describe state transitions.

### Q2. “When should I denormalize?”

When an important read is too expensive with normalized source data and the product can tolerate maintaining a derived copy. I explicitly identify the source of truth, update mechanism, lag, and repair path.

### Q3. “What is data ownership?”

The service/store responsible for authoritative mutation of a piece of business state. Clear ownership prevents conflicting writers and helps define transaction boundaries and event contracts.

### Q4. “Why include retention in modeling?”

Retention changes storage growth, partition design, deletion, backup policy, and compliance. Temporary data such as holds or ephemeral presence should not be modeled as permanent history unless required.

## Common mistakes to avoid

- Modeling only entities, not operations.
- One generic “data table” with no access strategy.
- Derived views with no authoritative source.
- Ignoring state transitions.
- Timestamp alone as unique global cursor.
- No deletion/tombstone strategy.
- Shared mutable ownership across many services.

## Short revision note

Data model = **entities + access patterns + invariants + ordering + ownership + lifecycle**. Database technology comes after this.

## Topics to revise

- [ ] access-pattern table
- [ ] entity ownership
- [ ] invariants
- [ ] state machines
- [ ] ordering/cursors
- [ ] denormalization
- [ ] source vs derived data
- [ ] retention/deletion

## Interview-ready synthesis

### A strong 60–90 second explanation

I model data from operations, not nouns alone. I list the main reads/writes, identify ownership/source of truth, define invariants and state transitions, choose ordering/pagination keys, and state retention. Derived caches/search/feed views are explicitly labeled rebuildable and may lag.

### How this topic connects to the wider system

- Correctness: invariants and state machines prevent invalid transitions.
- Performance: access patterns drive indexes and denormalized views.
- Scalability: shard keys should align with dominant access patterns.
- Maintainability: clear ownership prevents many services from mutating the same truth.

### Revision flashcards with answers

**Access pattern?**  
A concrete read/write the product performs, including key/filter/order/frequency.

**Invariant?**  
A rule that must always hold, such as unique username or no double booking.

**Derived view?**  
A copy/index/materialization that can be rebuilt from authoritative data.

**Why stable ID plus timestamp cursor?**  
Timestamps can collide; a tie-breaker gives deterministic ordering.

**Why model deletion?**  
Derived copies, retention, and tombstones make deletion a system-wide lifecycle.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- News feed: Post and Follow are source entities; per-user FeedEntry is a derived view.
- Ticketing: Seat/Hold/Order state machine matters more than a pretty ER diagram.
- File sync: stable file identity, versions, tombstones, and sync cursors are lifecycle modeling decisions.

### When not to overuse this idea

Do not choose a database before you can state the dominant reads/writes and invariants. Technology-first modeling usually creates awkward secondary queries later.

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

**Next:** Lesson 14 — NoSQL Databases
