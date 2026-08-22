# Session 17 — Consistent Hashing

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand the intuition behind distributing keys across a changing set of nodes with limited remapping.

## What You Need to Read / Learn

- Basic hash-based partitioning and why `hash(key) % N` remaps many keys when N changes.
- Consistent-hash ring intuition.
- Nodes and key positions on a ring.
- Clockwise ownership concept.
- Virtual nodes and why they improve balance/flexibility.
- Adding/removing nodes and limited key movement.
- Use in distributed caches and partition routing.
- Limitations: skew and operational complexity still exist.

## What You Need to Do

- [ ] Compare modulo hashing with consistent hashing when moving from 4 to 5 cache nodes.
- [ ] Draw a ring with 4 nodes and place 8 keys.
- [ ] Explain what happens when one node is removed.

## **Must Remember for the Interview**

- **Consistent hashing reduces remapping when the node set changes; it does not guarantee perfect balance by itself.**
- **Virtual nodes improve distribution and make capacity weighting easier.**
- **It is a partition-assignment technique, not a consistency model.**
- **Hot keys remain hot even with consistent hashing.**
- **Use it when membership changes and key movement cost matters.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Modulo hashing changes many assignments when N changes.**
- **Consistent hashing maps keys/nodes to a ring and moves only nearby ranges on membership changes.**
- **Virtual nodes smooth distribution.**
- **Still handle hotspots separately.**
- **Do not confuse consistent hashing with strong consistency.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain why modulo hashing is disruptive?
- [ ] Can I draw a consistent hash ring?
- [ ] Can I explain virtual nodes?
- [ ] Can I name one problem consistent hashing does not solve?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 17/46  
**Next:** Session 18
