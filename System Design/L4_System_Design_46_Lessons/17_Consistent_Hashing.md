# Session 17 — Consistent Hashing

## Outcome

You should understand why naive modulo hashing moves large amounts of data when nodes change, how a consistent-hash ring and virtual nodes reduce remapping, where it is useful, and what it does **not** solve.

## The Problem With Naive Modulo

Suppose:

```text
node = hash(key) mod 4
```

Add a fifth node:

```text
node = hash(key) mod 5
```

Most keys produce a different result. Huge remapping can:
- move data;
- invalidate caches;
- overload backing stores.

We want node changes to affect a smaller fraction of keys.

## Hash Ring Intuition

Map both:
- nodes;
- keys

onto a circular hash space.

```text
          Node A
       /          \
   keys            Node B
      \            /
        Node C
```

A key is assigned to the next node clockwise (one common convention).

When a node is added, it takes only the nearby range rather than changing mapping for almost every key.

When a node leaves, its range moves to its successor.

## Why It Helps

With many nodes and good distribution, adding/removing one node moves roughly that node's share of keys, not the majority.

Useful for:
- distributed caches;
- partition ownership;
- sharded stores;
- request routing by key.

## Virtual Nodes

One physical node owns many positions on the ring.

```text
A1, A2, A3 → physical Node A
B1, B2, B3 → physical Node B
```

Benefits:
- smoother distribution;
- easier weighting by capacity;
- when node fails, its ranges spread across multiple other nodes rather than one successor.

## Weighted Capacity

A larger node can own more virtual nodes/ranges.

But weights must be chosen carefully: CPU, memory, network, storage, and workload characteristics can differ.

## Replication Concept

A partition may be replicated to next N distinct physical nodes on ring.

Example:
- primary owner + two replicas.

This can provide availability, but now you must decide:
- consistency;
- quorum;
- failover;
- replica lag/conflicts.

Consistent hashing itself does not answer these.

## Cache Use Case

Imagine 100 application nodes choose among 20 cache nodes.

With naive modulo, adding cache node remaps most keys → enormous miss spike.

With consistent hashing, fewer keys move → smaller cold-cache event.

Client-side hashing is one pattern; proxy/service may also route.

## Hot Key Problem

If `celebrity:123` receives 1M RPS, consistent hashing maps it to one owner/range.

Adding nodes may not help.

Need separate mitigation:
- replicate hot value;
- local caches;
- CDN;
- key splitting if semantics allow;
- special routing.

**Consistent hashing balances many keys, not one extremely hot key.**

## Skewed Key Popularity

Uniform key distribution does not imply uniform traffic.

Example:
- 10M keys evenly distributed;
- top 100 keys produce 90% of requests.

Ring balance by key count can still yield traffic imbalance.

Monitoring real load matters.

## Node Churn

Frequent node changes cause:
- key movement;
- cache misses;
- replication work.

Use:
- stable membership;
- graceful rebalancing;
- virtual nodes;
- bounded data movement.

## Comparison With Directory-Based Routing

### Consistent hashing
Routing computed algorithmically from membership.

Pros:
- decentralized/simple mapping;
- limited movement.

### Directory
```text
partition42 → node7
```

Pros:
- explicit placement;
- move selected tenants/partitions;
- easier special handling.

Costs:
- directory is critical metadata;
- must be available/consistent.

Large real systems may use combinations/variants. At interview depth, choose based on need.

## Consistency Caveat

Name “consistent hashing” is unrelated to database **consistency** guarantee.

It is about stable key placement under membership changes.

Do not confuse with strong/eventual consistency.

## Worked Example — Distributed Cache

Requirements:
- 100M cache keys;
- 20 cache nodes;
- nodes can be added/removed;
- backend DB cannot tolerate 80% cache remap.

Design:
- hash key on ring;
- many virtual nodes per physical cache node;
- replicas for important cached data if justified;
- gradual warm/traffic shift.

Failure:
- one node dies → its owned ranges remap; DB sees misses only for those ranges rather than nearly all keys.

Still handle:
- hot keys;
- backend capacity;
- cold replacement;
- cache stampede.

## Small Design Drills

1. Why does `hash(key) mod N` cause large remapping when N changes?
2. What do virtual nodes improve?
3. Does consistent hashing make data strongly consistent?
4. Why can one celebrity key still overload one node?
5. What happens when a node leaves a ring?
6. When might directory-based routing be better?

<details>
<summary>Answer key</summary>

1. Modulus changes for almost every hash result, changing owner.
2. Distribution, weighting, and spreading failed-node ranges across physical nodes.
3. No. It is a placement strategy.
4. One key maps to limited owner(s); key popularity is skewed.
5. Its ranges transfer to successor/other owners depending on scheme.
6. Explicit tenant placement, targeted moves, compliance/isolation, special hotspots.

</details>

## Common Interview Mistakes

- Confusing consistent hashing with consistency models.
- Saying no data moves.
- Ignoring virtual nodes.
- Claiming it solves hot keys.
- Assuming equal key counts mean equal traffic.
- Ignoring cache miss burst during movement.
- Over-explaining ring math instead of design implications.

## Must Remember

- **Naive modulo remaps many keys when node count changes.**
- **Consistent hashing limits remapping to nearby ranges.**
- **Virtual nodes improve balance and weighting.**
- **It is useful for cache/partition ownership.**
- **It does not provide data consistency.**
- **It does not solve one hot key.**
- **Traffic skew can exist even with uniform key distribution.**
- **Directory routing is an alternative when explicit placement is valuable.**

## Interview Revision Summary

Use consistent hashing when:
```text
keys → changing node pool
```
and minimizing movement matters.

Then still ask:
```text
replication?
hot keys?
membership?
rebalancing?
backend miss capacity?
```

## Explain Without Notes

Explain why adding the 21st cache node to a 20-node modulo-hashed cluster can be dangerous and how consistent hashing changes the remapping behavior.

## Completion Checklist

- [ ] I understand modulo remapping.
- [ ] I can explain hash ring intuition.
- [ ] I understand virtual nodes.
- [ ] I know what consistent hashing does not solve.
- [ ] I can compare algorithmic vs directory routing.
