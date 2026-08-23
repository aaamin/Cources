# Lesson 18 — Consistency, CAP & Quorums

**Phase:** Fundamentals  
**Session:** 18/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn strong/eventual/read-after-write expectations, correct CAP intuition, quorum concepts, conflicts, and how to choose consistency per operation.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Strong vs eventual

Strong consistency gives reads a single coherent committed view under the chosen model. Eventual consistency allows temporary disagreement that converges. One system can use different guarantees for different data.

## Read-after-write and monotonic reads

Read-after-write means a successful writer should not immediately observe an older version. Monotonic reads mean once a client sees version V, later reads should not go backward. These user-visible guarantees are often more useful than broad labels.

## CAP

CAP matters when a network partition prevents replicas from communicating. During that partition you cannot provide both unrestricted availability and strong consistency for the affected data. CAP is not a static shopping label like “database X is AP forever.”

## Quorums

With N replicas, writes may wait for W and reads query R. The common overlap intuition `R + W > N` increases the chance that reads see a recent write. Real systems add versioning, repair, and failure details.

## Conflicts

Multi-writer or eventually consistent systems can accept concurrent conflicting updates. Resolution may use timestamps, application merge rules, versions, or rejection. Business semantics decide what is safe.

## Per-operation consistency

A like count can tolerate seconds of staleness; user settings often want read-after-write; seat reservation needs strong correctness. State the guarantee for the operation, not the whole product.

## Worked example — three feature guarantees

Like count: eventual is acceptable. Profile settings: read-after-write is desirable. Seat reservation: two users must not both confirm one seat, so stronger coordination is required. This illustrates why consistency is a product decision.

## Interview lens

Avoid reciting CAP theory. Translate consistency into user-visible behavior during failures and concurrent writes.

## What to remember

Use the weakest consistency that still preserves required business behavior. Stronger coordination has latency/availability cost.

## Review after reading

1. Eventual consistency allows what?
2. Read-after-write?
3. When is CAP relevant?
4. Quorum intuition?
5. One stale-tolerant and one strict feature?

## Deeper study notes

### Consistency has scope and time

Ask “consistent with respect to what?” Per-key linearizable reads, per-session read-after-write, per-conversation ordering, and globally serializable transactions are very different guarantees and costs. Avoid using “strong consistency” without naming the operation/scope.

### Availability is also scoped

During a partition, a system can keep reads available from stale replicas while rejecting conflicting writes. It can keep one region writable and make another read-only. CAP does not require one all-or-nothing product-wide choice.

### Quorums are not magic

`R + W > N` is a useful overlap intuition but assumes replicas and versions are managed appropriately. Sloppy quorums, hinted handoff, concurrent writes, and failures add nuance. For interviews, use quorum language to explain the latency/consistency trade-off, not to claim mathematical perfection.

### Eventual consistency requires convergence

“Eventually consistent” is not permission to ignore conflicts. The system needs a mechanism that causes replicas/derived views to converge: last-write rule, version-based reconciliation, stream replay, anti-entropy, or application merge.

### Common mistakes

- Saying CAP means choose only two of C/A/P in normal operation.
- Demanding strong consistency for like counts or presence with no business reason.
- Using eventual consistency for money/inventory without explaining conflict handling.
- Assuming timestamps always resolve concurrent writes correctly despite clock skew.


## Important interview ideas

> **Important:** Do not say “the system is strongly consistent” without specifying **which operation, which data, and which failure condition**. Consistency should be scoped.

### Useful consistency guarantees

Think in product behavior:

- **read-after-write:** after I save my profile, I see the new value;
- **monotonic reads:** once I have seen version 10, I do not later see version 9;
- **per-key/linearizable operation:** concurrent updates behave as one ordered current value;
- **transactional consistency:** a set of invariants changes atomically;
- **eventual consistency:** replicas/derived views may lag but converge.

These are more useful than repeating “strong vs eventual” without context.

### CAP as a failure decision

CAP becomes interesting when nodes cannot communicate because of a network partition. Suppose two regions both have copies of inventory.

During partition:

- allow both to sell → high availability, possible oversell/conflict;
- keep only one region writable or reject uncertain writes → preserve correctness, lower availability.

That is the practical CAP conversation. Under normal operation, the system may offer both low latency and consistent behavior; the trade is exposed by partition.

### Quorum intuition

For `N=3`, a system might require `W=2` replicas for a write and `R=2` for a read. Read and write sets overlap, so a read can observe at least one copy containing the latest acknowledged version, assuming the version-resolution model supports it.

Quorum does not eliminate all distributed-system nuance. Concurrent writes, stale nodes, hinted handoff, and repair still matter.

### Conflict resolution

If multiple writers are accepted, decide how conflicts converge:

- last-write-wins (simple, can lose updates, depends on clocks/versioning);
- application merge (e.g., set union);
- version vectors/vector clocks conceptually;
- route all writes for one key to one leader;
- reject conflict and ask client to retry.

Choose based on semantics. You cannot safely merge “seat sold to Alice” and “seat sold to Bob.”

## Consistency decision examples

| Data | Likely requirement | Reason |
|---|---|---|
| presence | eventual | brief stale online status is acceptable |
| like count | eventual | exact instant count rarely critical |
| own profile update | read-after-write | UX expectation |
| password/security setting | strong/read-after-write | security correctness |
| seat reservation | strong invariant | no double sale |
| feed ranking | eventual | derived view |

## Interview questions and model answers

### Q1. “What does CAP mean?”

When a network partition prevents replicas from communicating, a distributed system cannot guarantee both full availability and strong consistency for the affected state. It must reject/delay some operations or accept divergent results. I would explain the choice for the specific operation.

### Q2. “Why not use strong consistency everywhere?”

Coordination costs latency and can reduce availability during failures. Many product features do not need it. Using weaker consistency for derived/low-risk data improves scale and resilience while strong guarantees remain around critical invariants.

### Q3. “What is eventual consistency?”

Replicas or derived views can temporarily differ, but the system has a convergence mechanism and, absent new writes, eventually reaches a common state. It does not mean “we ignore inconsistencies.”

### Q4. “How do quorums help?”

By requiring overlapping subsets of replicas for reads and writes, the system can improve probability/guarantee of seeing recent data depending on its version model. Larger quorums improve consistency/durability but cost latency and availability.

## Common mistakes to avoid

- “Pick two of CAP” without partition context.
- One consistency setting for every feature.
- Eventual consistency with no convergence mechanism.
- Last-write-wins used for non-mergeable business conflicts.
- Strong global coordination for harmless counters/presence.
- Quorum formula presented as solving every conflict.

## Short revision note

Ask: **what data, what operation, what user-visible guarantee, and what happens during partition?** Use strong consistency only where business correctness demands it.

## Topics to revise

- [ ] strong vs eventual
- [ ] read-after-write
- [ ] monotonic reads
- [ ] CAP under partition
- [ ] quorum R/W/N intuition
- [ ] conflict resolution
- [ ] convergence
- [ ] operation-specific consistency

## Interview-ready synthesis

### A strong 60–90 second explanation

I specify consistency per operation. I translate CAP into a concrete partition-time decision, use read-after-write/monotonic guarantees where useful, and only use strong coordination for invariants that cannot tolerate conflicts. Quorum is a latency/availability/consistency tool, not a magic formula.

### How this topic connects to the wider system

- Correctness: consistency level must preserve business invariant/user expectation.
- Availability: stronger coordination may reject/delay operations during partitions.
- Performance: quorum/cross-region coordination costs latency.
- Scalability: eventual derived views reduce synchronous coordination.

### Revision flashcards with answers

**Read-after-write?**  
A client that successfully writes should not immediately read an older value.

**CAP applies when?**  
A network partition prevents distributed replicas from communicating.

**Eventual consistency requires?**  
A convergence/reconciliation mechanism, not just tolerance of disagreement.

**Quorum intuition?**  
Overlapping read/write replica sets can expose recent versions.

**Like count vs seat?**  
Like count can often lag; seat confirmation requires strong invariant.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Chat presence: eventual is fine; accepted message durability needs stronger guarantees.
- Feed materialization: derived feed can lag and replay; post source stays authoritative.
- Ticket inventory: availability may be sacrificed during partition to prevent conflicting confirmed sales.

### When not to overuse this idea

Do not optimize consistency as a badge. Stronger is not automatically better; use the minimum guarantee that preserves business/user semantics.

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

**Next:** Lesson 19 — Message Queues & Async Processing
