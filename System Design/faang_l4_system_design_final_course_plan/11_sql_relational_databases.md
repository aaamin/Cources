# Session 11 — SQL & Relational Databases

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Build a strong default mental model for structured data, constraints, joins, and transactions.

## What You Need to Read / Learn

- Tables, rows, columns, primary keys, foreign keys.
- One-to-one, one-to-many, and many-to-many relationships.
- Normalization and why it reduces duplication.
- Selective denormalization for read performance.
- ACID transaction intuition.
- Constraints: uniqueness, foreign keys, checks, not-null.
- Joins and why relationship-heavy workloads fit relational systems naturally.
- Durability and write-ahead logging at recognition depth only.
- When a relational database is the simplest correct default.

## What You Need to Do

- [ ] Model users, products, orders, order_items, payments, and inventory.
- [ ] List the invariants the database can enforce directly.
- [ ] Identify one place where denormalization may improve a frequent read.

## **Must Remember for the Interview**

- **Choose a relational database when transactions, constraints, and relationships are important unless scale/access patterns force another choice.**
- **Normalization improves integrity; denormalization can improve reads at the cost of write/consistency complexity.**
- **Database constraints are often safer than relying only on application checks.**
- **A transaction groups changes into an atomic correctness boundary.**
- **SQL vs NoSQL is a workload decision, not a scalability slogan.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Relational DB strengths: constraints, transactions, joins, mature indexing.**
- **Model invariants explicitly.**
- **Use denormalization intentionally and know how derived copies stay correct.**
- **Start with the simplest storage model that meets requirements.**
- **Strong schema can be an advantage, not a limitation.**

## Self-Test Before Marking This Session Complete

- [ ] Can I model a many-to-many relationship?
- [ ] Can I explain normalization vs denormalization?
- [ ] Can I state an invariant a DB constraint can enforce?
- [ ] Can I explain why SQL can still scale?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 11/46  
**Next:** Session 12
