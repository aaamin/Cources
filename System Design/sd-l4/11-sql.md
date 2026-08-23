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
