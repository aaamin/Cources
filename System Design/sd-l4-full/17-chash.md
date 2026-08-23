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


## Important interview ideas

> **Important:** Consistent hashing is primarily a **membership-change** solution. It reduces remapping when cache/storage nodes join or leave. It does not automatically solve replication, hot keys, or data correctness.

### Why modulo hashing hurts

With ordinary modulo routing:

```text
owner = hash(key) % number_of_nodes
```

changing the node count changes the divisor. A large fraction of keys now map elsewhere. In a distributed cache, this creates a huge cold-cache event. In storage, it can trigger massive data movement.

Consistent hashing decouples key placement from a simple node count. Keys and node positions share one hash space, and a node change affects only adjacent ranges.

### Virtual nodes and balance

One physical server can own many virtual positions. This helps because a small number of physical nodes might otherwise receive uneven hash intervals. With many virtual nodes, ownership can be spread more smoothly, and more powerful machines can receive more virtual nodes.

Virtual nodes do not prevent **application-level** skew. One celebrity key still maps to one ownership set unless the system explicitly replicates/splits it.

### Replication on a ring

A storage system can replicate a key to the next few distinct nodes around the ring:

```text
primary owner + replica 1 + replica 2
```

This gives redundancy, but replication semantics—write acknowledgement, repair, consistency—are separate design decisions. The ring only provides placement.

### Membership/control plane

Routers must know which nodes currently own which ranges. Real systems distribute membership via a control plane, gossip, or configuration service. During transitions, versions/epochs prevent stale routers or owners from corrupting placement.

At L4, state this conceptually; do not implement membership consensus unless asked.

## Alternatives

| Technique | Strength | Trade-off |
|---|---|---|
| modulo hash | extremely simple | huge remap on N change |
| consistent hash | localized movement | ring/membership complexity |
| rendezvous hash | simple stable ranking | different routing mechanics |
| logical shards + shard map | explicit movement/control | central/control metadata |

Interviewers usually care that you understand the problem, not that every system uses a ring.

## Worked scenario — cache node failure

Ten cache nodes hold millions of keys. One node fails.

With consistent hashing, requests for ranges owned by that node move to neighboring/new owners. If cache data is disposable, misses refill from the source. If this were durable storage, replicas would take over and repair the lost copy.

The key operational concern is avoiding a sudden overload on successor nodes. Virtual nodes and replica distribution spread takeover more evenly.

## Interview questions and model answers

### Q1. “Why is consistent hashing useful for caches?”

Because cache nodes can be added/removed and we want most keys to stay on the same owner. Stable placement preserves hit rate and avoids a full cluster cold start.

### Q2. “Does it guarantee equal load?”

No. It improves hash-space distribution, especially with virtual nodes, but real traffic can still be skewed by hot keys or tenants. Load balancing may require replication, local caching, or splitting hot state.

### Q3. “Do I need consistent hashing to shard a database?”

Not necessarily. Many databases use fixed logical shards and a routing map, or manage partitions internally. Consistent hashing is one routing strategy. I would use the simplest approach that supports rebalancing and ownership changes.

### Q4. “What happens when a node joins?”

It takes ownership of selected hash ranges from existing nodes. Only keys in those ranges move. The system transfers/warmups data and updates membership/routing metadata.

## Common mistakes to avoid

- “Consistent hashing solves hot keys.”
- Ring diagram with no membership-change explanation.
- Forgetting replication/durability.
- Assuming perfect balance with few nodes.
- Using it when fixed shard mapping is simpler.

## Short revision note

Consistent hashing = **stable key placement under changing membership**. Virtual nodes improve balance; replication and hot-key handling are separate.

## Topics to revise

- [ ] modulo remapping problem
- [ ] hash ring
- [ ] virtual nodes
- [ ] range ownership
- [ ] replication placement
- [ ] membership/control plane
- [ ] hot-key limitation
- [ ] alternatives such as shard maps

## Interview-ready synthesis

### A strong 60–90 second explanation

I explain consistent hashing as stable placement when membership changes. Keys and nodes share hash space; only adjacent ranges move when nodes join/leave. Virtual nodes improve distribution, while replication and hot-key mitigation remain separate concerns.

### How this topic connects to the wider system

- Scalability: nodes can be added without remapping the full key space.
- Reliability: ownership changes after node failure; replicas must still provide durability.
- Performance: fewer cache keys move, preserving hit rate during resize.
- Operations: routers require a consistent membership/ring view.

### Revision flashcards with answers

**Modulo problem?**  
Changing N remaps a large fraction of `hash(key)%N` keys.

**Virtual node?**  
Multiple logical ring positions owned by one physical server to improve balance.

**Hot key solved?**  
No; one key still has concentrated traffic unless replicated/split.

**Does ring replicate?**  
Not inherently; replication policy is separate.

**Alternative?**  
Logical shards with an explicit shard map or rendezvous hashing.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Distributed cache: stable ownership preserves hit rate when nodes scale in/out.
- Dynamo-style storage: ring placement can choose primary/replica owners, with separate quorum/repair logic.
- Request routing: shard ownership can move incrementally rather than remapping every key when capacity changes.

### When not to overuse this idea

Do not mention consistent hashing in a fixed database topology if a simple shard map explains ownership more clearly.

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

**Next:** Lesson 18 — Consistency, CAP & Quorums
