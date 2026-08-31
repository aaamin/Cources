# Session 12 — Database Indexing

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand how indexes make specific queries faster and what they cost on writes and storage.

## What You Need to Read / Learn

- Primary/clustered index concepts and secondary indexes.
- Composite indexes and left-prefix intuition.
- Covering index concept: query can be answered from index entries without fetching full row.
- Selectivity/cardinality.
- Equality versus range queries.
- Sort order and indexes.
- Index maintenance cost on writes.
- Too many indexes and storage/write amplification.
- Query patterns before index design.

## What You Need to Do

- [ ] Given `posts(user_id, created_at, status, id)`, design an index for newest posts by user.
- [ ] Given an orders table, choose indexes for order-by-id, customer history, and pending orders by time.
- [ ] Explain why an index on a low-cardinality boolean field may have limited value.

## **Must Remember for the Interview**

- **Indexes accelerate particular access patterns; they are not free.**
- **Composite index column order matters.**
- **Every index consumes storage and adds write/update work.**
- **Design indexes from real queries, not from columns that 'look important'.**
- **A query can still be slow if it returns a huge result set even with an index.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Access pattern → index; not index → hope.**
- **Composite index order should match equality/range/sort needs.**
- **Indexes trade write cost/storage for read performance.**
- **Primary and secondary indexes serve different access paths.**
- **Mention the expected query when proposing an index in an interview.**

## Self-Test Before Marking This Session Complete

- [ ] Can I design a composite index for a concrete query?
- [ ] Can I explain write amplification from indexes?
- [ ] Can I explain index selectivity?
- [ ] Can I identify when an index will not solve the problem?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 12/46  
**Next:** Session 13
