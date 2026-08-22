# Session 13 — Data Modeling & Access Patterns

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn to derive schema and ownership from how the system reads/writes data, rather than picking a database first.

## What You Need to Read / Learn

- Identify entities and relationships.
- List critical read/write access patterns before schema design.
- Define ownership: which service/table/store is authoritative for each entity.
- Choose identifiers and uniqueness rules.
- Identify ordering requirements and natural partition keys.
- Identify retention/deletion requirements.
- Decide which derived views may be denormalized or cached.
- Map consistency needs to operations.
- Avoid designing one schema for every hypothetical query.

## What You Need to Do

- [ ] Model a social system with User, Post, Follow, Like, Comment after listing access patterns.
- [ ] Model a chat system around conversations and messages. Compare partition-by-user vs partition-by-conversation.
- [ ] For each model, state source of truth and two derived views.

## **Must Remember for the Interview**

- **Access patterns should drive schema, indexes, partitioning, and often database choice.**
- **State the source of truth and ownership explicitly.**
- **Different data can require different consistency guarantees within one system.**
- **Derived views improve reads but create update/reconciliation work.**
- **Data modeling is part of system design, not a separate afterthought.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **First ask: What are the important reads/writes?**
- **Then define entities, ownership, keys, indexes, retention, consistency.**
- **Only then choose storage technology.**
- **Partition key choices should follow dominant access paths and skew.**
- **Explain how derived/denormalized data stays synchronized.**

## Self-Test Before Marking This Session Complete

- [ ] Can I list access patterns before drawing tables?
- [ ] Can I identify the source of truth?
- [ ] Can I distinguish authoritative data from derived data?
- [ ] Can I choose a natural partition key from access patterns?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 13/46  
**Next:** Session 14
