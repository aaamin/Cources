# Lesson 17 — Consistent Hashing

**Phase:** Fundamentals  
**Session:** 17/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn why modulo hashing remaps heavily when node count changes, how a hash ring limits movement, and how virtual nodes improve balance.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Modulo remapping

`hash(key) % N` works while N stays fixed. Changing from N=10 to N=11 changes the modulo for many keys, causing widespread movement and cold cache/storage rebalancing.

## Hash ring

Consistent hashing maps nodes and keys into one circular hash space. A key belongs to the next node clockwise. When one node joins/leaves, only neighboring ranges move.

## Virtual nodes

A physical machine can own many positions on the ring. Virtual nodes smooth distribution and let stronger machines own more ranges.

## Use cases

Distributed caches, storage ownership, and request routing can benefit when membership changes. Managed systems may hide the mechanism, so use it when the conceptual problem actually exists.

## Limitations

Consistent hashing reduces movement but does not solve a single hot key. That key still maps to an owner unless replicated or split. Membership and replication logic remain necessary.

## Worked example — expanding a cache cluster

With modulo hashing, adding a fifth node remaps a large fraction of cache keys. With consistent hashing, the new node takes selected ring ranges from neighbors, so most keys stay where they were and the cache remains warmer.

## Interview lens

Use consistent hashing when dynamic membership/data ownership is relevant. Do not insert it into every sharding design.

## What to remember

Consistent hashing is about **minimizing key movement when nodes change**; virtual nodes help distribution, but hot keys remain a separate problem.

## Review after reading

1. Why does modulo remap?
2. What is the ring?
3. Why virtual nodes?
4. Natural use case?
5. What does it not solve?

## Deeper study notes

### Think of ownership intervals

The ring is easiest to reason about as ownership of intervals in hash space. Each node owns one or more intervals. Adding a node transfers selected intervals to it. Removing a node transfers its intervals to successors or replicas. The benefit is localized membership change.

### Replication often follows the ring

A storage system may place a key on its primary ring owner plus the next `r-1` distinct nodes. Node failure then changes which replicas serve the range. Consistent hashing by itself does not provide durability; the replication rule does.

### Membership needs agreement

Every client/router must have a sufficiently consistent view of which nodes own which ranges. Real systems use configuration services, gossip, or control planes. If different routers disagree badly, requests can be misrouted. In interviews, you can abstract this as a membership/control-plane service.

### Alternatives are valid

Rendezvous hashing, fixed logical shards with a shard map, and database-managed partitioning solve similar routing problems. Consistent hashing is useful knowledge, not the only acceptable answer.

### Common mistakes

- Drawing a ring without explaining why node changes matter.
- Claiming it guarantees perfect load balance.
- Forgetting replication.
- Using it when shard membership is static and a simple shard map is clearer.


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

**Next:** Lesson 18 — Consistency, CAP & Quorums
