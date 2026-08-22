# Session 14 — NoSQL Databases

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand major non-relational storage families and when their access patterns are a better fit than a relational model.

## What You Need to Read / Learn

- Key-value stores: key-based lookup, simple values, horizontal distribution.
- Document stores: document-shaped records, flexible nested data, indexed fields.
- Wide-column stores: large partitioned datasets organized around partition/clustering keys.
- Denormalization and query-first modeling.
- Limited joins and cross-record transactions depending on system.
- Partition-key importance and hot-partition risk.
- Horizontal scaling and availability trade-offs.
- When NoSQL is unnecessary.

## What You Need to Do

- [ ] Choose a storage family for user sessions, product catalog documents, high-volume telemetry, and payment ledger.
- [ ] For each choice, state one alternative and why it is weaker for the stated workload.
- [ ] Design a wide-column-style key for messages by conversation and time.

## **Must Remember for the Interview**

- **NoSQL is not one thing; key-value, document, and wide-column stores optimize different access patterns.**
- **NoSQL does not mean 'no schema'; the schema often lives in application/data-access design.**
- **Partition-key choice can dominate scalability.**
- **Denormalization improves local reads but makes updates/reconciliation harder.**
- **Do not choose NoSQL merely because the system is large.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Key-value → direct key lookup. Document → document-centric data. Wide-column → partitioned high-scale access patterns.**
- **Start from queries and consistency requirements.**
- **Know the cost of joins/transactions you give up or complicate.**
- **Hot partitions are a common failure mode.**
- **SQL remains a valid choice at large scale.**

## Self-Test Before Marking This Session Complete

- [ ] Can I differentiate key-value, document, and wide-column?
- [ ] Can I give a reason not to use NoSQL?
- [ ] Can I explain denormalization cost?
- [ ] Can I choose a partition key for a NoSQL table?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 14/46  
**Next:** Session 15
