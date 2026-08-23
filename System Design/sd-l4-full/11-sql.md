# Lesson 11 — SQL & Relational Databases

**Phase:** Fundamentals  
**Session:** 11/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn relational tables, keys, relationships, constraints, normalization, transactions, and why SQL is often a strong default when correctness and flexible queries matter.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Relational model

Relational databases organize data into tables with typed columns and rows. Relationships are expressed through keys. Example: `User(user_id, name)` and `Order(order_id, user_id, status)`. The foreign-key relationship lets the database reason about integrity between the records.

## Primary and foreign keys

A primary key uniquely identifies a row. A foreign key references another row/table and can enforce referential integrity. Constraints allow the database to protect some correctness rules instead of relying entirely on application code.

## Normalization

Normalization stores a fact in one logical place to reduce duplication and update anomalies. Instead of copying a user's email into every order, store the user once and reference the ID. Denormalization intentionally duplicates selected data for faster reads, at the cost of synchronization.

## Transactions

A transaction groups related operations under atomicity/isolation rules. If an operation must update several rows while preserving an invariant, a relational transaction is often the simplest correctness tool when the data is colocated.

## Constraints and invariants

Unique constraints, foreign keys, checks, and transactional updates can enforce rules such as unique username, non-negative balances, or one seat assignment. Critical invariants are safer when the storage layer participates in enforcing them.

## Why SQL can still scale

Do not equate “large system” with “NoSQL.” Relational databases support indexing, replication, partitioning, and managed scale. The simplicity of transactions and flexible queries can outweigh premature distribution complexity.

## Worked example — e-commerce order model

Use `User`, `Order`, `OrderItem`, and `Payment` tables. Enforce valid relationships and transactionally protect state transitions. If scale later demands partitioning, the model can evolve; first make business correctness understandable.

## Interview lens

In interviews, SQL is often a reasonable starting point. Explain when relationships, transactions, constraints, and flexible queries matter before deciding to move away from it.

## What to remember

Relational databases are powerful because they combine queryability with mature integrity and transaction mechanisms. Scale alone is not a reason to reject them.

## Review after reading

1. What does a primary key guarantee?
2. Why use foreign keys?
3. Normalization vs denormalization?
4. What does a transaction help preserve?
5. Give one invariant a DB can enforce.

## Deeper study notes

### ACID at interview depth

Atomicity means a transaction's changes commit together or not at all. Consistency means transactions preserve declared invariants. Isolation controls what concurrent transactions observe. Durability means committed data survives failures within the database's guarantee. You do not need to recite every isolation anomaly unless concurrency is central, but know why transactions simplify state changes.

### Normalization is not a moral rule

Normalized schemas reduce duplicate facts and anomalies. High-scale read paths may intentionally denormalize. The question is who owns the canonical fact and how copies are updated. If every order needs the historical product price, copying the price into `OrderItem` is correct because that is an immutable business fact for the order, not accidental duplication.

### Transaction boundaries should follow invariants

If two rows must change together, keeping them in one database/partition can be valuable. Splitting every entity into a separate microservice can turn a simple transaction into a saga. Service boundaries therefore affect data correctness cost.

### SQL query flexibility has value

Product requirements change. Relational systems allow new joins and indexes without redesigning the whole physical schema. Query-shaped NoSQL schemas can be faster for known paths but more expensive to evolve when access patterns change.

### Common mistakes

- Choosing NoSQL solely because user count is large.
- Treating foreign keys as mandatory in every high-scale schema; sometimes application ownership replaces them.
- Over-normalizing a hot read path and creating many joins without measuring.
- Splitting one invariant across services unnecessarily.


## Important interview ideas

> **Important:** A relational database is not “small scale.” It is often the best starting point because it gives strong integrity, flexible queries, and mature transactions. Distribution should be justified by actual scale or availability requirements.

### Constraints are architecture tools

If an invariant can be enforced by the database, do not rely only on application convention.

Examples:

```sql
UNIQUE(username)
FOREIGN KEY(order.user_id)
CHECK(amount >= 0)
```

For booking, a unique/conditional constraint can prevent two confirmed records for the same logical resource. Database constraints protect against races across multiple application instances.

### Normalization vs denormalization

Normalization stores a fact once. This reduces update anomalies and keeps truth clear. Denormalization stores copies to make a read cheaper.

A useful rule:

> Normalize the source of truth; denormalize selectively for proven read paths.

Derived copies need update/repair semantics. If a user's display name is copied into millions of historical posts, changing it may require either asynchronous propagation or accepting historical snapshots.

### Transactions and boundaries

Transactions are easiest when all required state lives in one database. If Order and Inventory are in the same relational store, an atomic transaction can update both.

Splitting them into separate services/databases turns one local transaction into a distributed workflow requiring Saga/outbox/reconciliation. Microservice boundaries therefore have correctness cost.

### Isolation at interview depth

You do not need to memorize every SQL isolation anomaly, but know that concurrent transactions can race. Techniques include:

- row locks/pessimistic locking;
- optimistic version checks;
- serializable transactions for narrow critical sections;
- unique constraints/conditional writes.

Pick the mechanism based on contention and invariant.

## Worked scenario — order creation

A simple relational model:

```text
orders(id, user_id, status, total)
order_items(order_id, product_id, qty, price)
payments(id, order_id, provider_ref, status)
```

Critical query patterns:

- get order by ID;
- list orders for user by created time;
- find payment by provider reference;
- transition order state.

Indexes follow those queries. Transactions protect state transitions. This is a complete reasoning chain before any discussion of sharding.

## Interview questions and model answers

### Q1. “SQL or NoSQL for this system?”

I first list access patterns, relationships, and invariants. If the data is relational and transactions/uniqueness matter, SQL is my default. I move to a distributed NoSQL model when predictable key-based access and horizontal distribution provide clear benefit that outweighs join/transaction limitations.

### Q2. “Is normalization always better?”

No. Normalize for correctness and clear ownership, then denormalize read-heavy derived views when joins or fan-out become too expensive. The denormalized copy must have a source of truth and repair strategy.

### Q3. “Can SQL scale horizontally?”

Yes—through sharding, distributed SQL, read replicas, and partitioning. The phrase “SQL does not scale” is outdated. The question is whether the chosen database and workload can scale while preserving required semantics at acceptable complexity/cost.

### Q4. “Why use a DB constraint if application checks already exist?”

Application checks can race across instances. A unique constraint or transactional conditional update is enforced at the authoritative state boundary and provides a final correctness guarantee.

## Common mistakes to avoid

- “NoSQL because millions of users.”
- No database constraints for critical uniqueness.
- Over-normalizing hot read paths without considering query cost.
- Denormalizing without a source-of-truth/repair plan.
- Breaking one transactional domain into many services too early.
- Designing tables before listing access patterns.

## Short revision note

Relational strength = **transactions + constraints + relationships + flexible queries**. Start from invariants/access patterns, not database fashion.

## Topics to revise

- [ ] primary/foreign keys
- [ ] unique/check constraints
- [ ] normalization
- [ ] denormalization
- [ ] transactions
- [ ] concurrency control
- [ ] transaction boundaries
- [ ] SQL as default vs NoSQL trigger

## Interview-ready synthesis

### A strong 60–90 second explanation

I use relational storage when relationships, constraints, and transactions are valuable. I identify invariants and let the database enforce them where possible. I normalize authoritative data for clarity, then denormalize selected read paths only when needed. Splitting one transactional domain into multiple databases has a correctness cost.

### How this topic connects to the wider system

- Correctness: constraints and transactions enforce invariants across concurrent app instances.
- Performance: indexes/denormalization optimize important queries.
- Scalability: SQL can use replicas/partitioning; scale alone does not imply NoSQL.
- Architecture: service boundaries should respect transaction/ownership boundaries where practical.

### Revision flashcards with answers

**Primary key?**  
Unique row identity and primary lookup path.

**Foreign key?**  
A constraint referencing another row/table to preserve relationships.

**Normalize?**  
Store facts once to reduce update anomalies.

**Denormalize?**  
Duplicate derived data to accelerate a read, accepting synchronization cost.

**Why DB constraint?**  
Application checks can race; authoritative constraints protect correctness.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Ticket booking: relational conditional updates/transactions naturally protect seat/hold invariants.
- E-commerce: normalized order/order-item/payment source data supports audit and correctness; read views can be derived later.
- User accounts: unique email/username constraints provide authoritative race-safe uniqueness.

### When not to overuse this idea

Do not split tightly transactional tables into separate databases unless service ownership/scale benefits outweigh the need for Saga/reconciliation.

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

**Next:** Lesson 12 — Database Indexing
