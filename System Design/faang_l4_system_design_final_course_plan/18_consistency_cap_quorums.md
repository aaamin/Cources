# Session 18 — Consistency, CAP & Quorums

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn to state the consistency guarantee an operation needs and explain availability trade-offs during partitions.

## What You Need to Read / Learn

- Strong consistency and linearizable intuition.
- Eventual consistency.
- Read-after-write consistency.
- Monotonic-read intuition.
- CAP theorem framed specifically around network partitions.
- Consistency versus availability choice during partition.
- Quorum reads/writes and `R + W > N` intuition.
- Version/conflict resolution at recognition depth.
- Different consistency requirements for different data in the same system.

## What You Need to Do

- [ ] Assign consistency requirements to likes, feed entries, account balances, seat reservations, profile settings, and presence.
- [ ] Explain a network partition between two regions and what a CP-style versus AP-style service might do.
- [ ] Work through N=3 replicas with different R/W quorum choices.

## **Must Remember for the Interview**

- **CAP is about behavior during a network partition; it is not 'pick any two forever'.**
- **Strong consistency usually costs latency and/or availability.**
- **Eventual consistency is acceptable only when temporary divergence is acceptable to the product.**
- **Quorums are one mechanism, not a universal magic formula.**
- **State consistency per operation, not just per system.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Ask: What user-visible anomaly is acceptable?**
- **Seat/payment correctness usually needs stronger guarantees than likes/presence.**
- **During partition, a system may reject operations to preserve consistency or accept divergence to preserve availability.**
- **Quorum intuition: overlapping read/write sets can improve freshness.**
- **Be precise about consistency instead of saying 'CAP says choose CP'.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain CAP accurately in one minute?
- [ ] Can I give examples of strong vs eventual consistency?
- [ ] Can I explain quorum intuition?
- [ ] Can I choose consistency per operation?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 18/46  
**Next:** Session 19
