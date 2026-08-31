# Session 18 — Consistency, CAP & Quorums

## Outcome

You should be able to define strong, eventual, read-after-write, and monotonic/session consistency at interview depth; explain CAP correctly during network partitions; reason about read/write quorums; and assign consistency requirements per operation rather than labeling an entire system vaguely.

## What Does “Consistent” Mean Here?

In distributed systems, consistency describes what values reads may observe relative to writes.

This is different from ACID “Consistency,” which is about preserving database invariants.

## Strong Consistency

After a successful write, subsequent reads behave as though there is one up-to-date logical copy according to the system's guarantee.

Useful for:
- seat ownership;
- unique reservation;
- account transfer decisions;
- configuration where stale values are unsafe.

Costs can include:
- coordination;
- leader/quorum dependency;
- higher latency;
- reduced availability under partition.

## Eventual Consistency

Replicas may temporarily disagree, but if writes stop, they converge eventually.

Useful for:
- like/view counters;
- search index;
- feed propagation;
- analytics.

“Eventual” is not a latency guarantee. It does not mean “within 2 seconds” unless product defines/bounds it.

## Read-After-Write Consistency

User should see their own successful write.

Example:
```text
update profile
immediately open profile
```
should show new value.

Other users may still see old value.

This can be achieved by:
- reading leader after write;
- session routing;
- consistency token/version;
- sufficiently synchronous replication.

Very common UX requirement.

## Monotonic Reads

Once a client has seen version 10, it should not later see version 9.

Without it:

```text
read replica A → v10
next read replica B → v9
```

The experience moves backward.

Sticky/session-aware routing or version tokens can help.

## Session Consistency

Broad concept: provide guarantees within one client's session while allowing weaker global consistency.

This often gives good UX without global strong coordination.

## Consistency Is Per Operation

Example social app:

| Operation | Possible requirement |
|---|---|
| Like count | eventual |
| User sees own post | read-after-write |
| Username uniqueness | strong invariant |
| Feed ranking | eventual |
| Account deletion permission | strong/fresh |

Do not say:
> “Instagram is eventually consistent.”

Say what can be stale and what cannot.

## CAP Theorem — Correct Mental Model

When a **network partition** divides nodes, a distributed system must choose how to behave for operations that require communication.

- **Consistency (C)**: maintain a single consistent view/linearizable-like guarantee for the relevant operation.
- **Availability (A)**: every request to a non-failed node receives a non-error response.
- **Partition tolerance (P)**: system continues operating despite network communication loss between parts.

Real distributed systems cannot prevent partitions, so during a partition the interesting trade-off is often:

```text
preserve consistency and reject/delay some requests
            vs
remain available and risk divergent/stale state
```

CAP is **not** “choose any two at all times.”

Outside a partition, systems can often provide both strong consistency and availability under normal conditions.

## Example — Inventory During Partition

Two regions cannot communicate. One item remains.

If both regions continue selling independently:
- availability high;
- may oversell.

If one region must contact authoritative leader/quorum and fails closed:
- preserve correctness;
- some users cannot buy.

Product invariant decides.

## Quorums

Suppose N replicas.

Write to W replicas.
Read from R replicas.

Simple quorum intuition:

```text
R + W > N
```

creates overlap between read and write sets.

Example:
```text
N=3, W=2, R=2
```

Any read quorum overlaps any successful write quorum.

But this alone does not magically give perfect strong consistency. Real systems also need:
- versioning;
- latest-value selection;
- concurrent write conflict handling;
- timing/repair semantics.

Use quorum math as intuition, not a universal proof.

## Quorum Trade-Offs

High W:
- slower/more fragile writes;
- fresher replicated state.

Low W:
- faster/more available writes;
- more stale/conflict risk.

High R:
- more read coordination/latency;
- better chance to observe latest.

Low R:
- faster reads;
- more stale risk.

Tune by workload/guarantee.

## Read Repair and Anti-Entropy Recognition

Leaderless systems may discover replicas disagree.

Conceptually:
- read multiple versions;
- select/merge latest;
- repair stale replica;
- background synchronization.

Recognition-level only.

## Conflict Resolution

When concurrent writes occur, possible strategies:

### Last-write-wins
Simple but can silently discard update; depends on timestamps/clock semantics.

### Version/vector-style metadata
Detect concurrent versions; application merges.

### Domain merge
Shopping cart might union items, but quantity conflicts still need rules.

### Single-writer/home region
Avoid conflicts rather than resolve them.

Often avoidance is simpler for correctness-critical data.

## Stale Reads

Ask:
- how stale?
- for whom?
- can stale read cause wrong write?
- can client retry?
- does user need monotonic/read-own-write?

A 5-second stale product description is different from a 5-second stale seat reservation.

## Worked Example — Social Post

Requirements:
- author sees new post immediately;
- followers may see it within seconds;
- like count may lag;
- deletion should propagate quickly, especially for privacy.

Design:
- post source stored durably;
- author's read routed to fresh source/read-after-write;
- feed fan-out eventually consistent;
- like count asynchronously aggregated;
- deletion event invalidates derived caches/feed/search, with stronger handling for access authorization.

One system, multiple consistency requirements.

## Small Design Drills

1. What does eventual consistency guarantee?
2. What does CAP actually force during network partition?
3. Why is “choose any two of CAP” misleading?
4. N=5, W=3, R=3: do read/write sets overlap?
5. Why can last-write-wins be dangerous?
6. Give one read-after-write user experience.
7. Should like count and seat booking have same consistency?

<details>
<summary>Answer key</summary>

1. Replicas converge if updates stop, according to system's conflict/replication semantics; it does not by itself bound delay.
2. For operations spanning partition, preserving strong consistency may require rejecting/delaying requests, while serving both sides may sacrifice consistency.
3. P is a fault condition you must tolerate; trade-off becomes relevant during partitions, not a permanent menu of two properties.
4. Yes, 3+3>5.
5. Concurrent valid updates can be lost; clock ordering can also be problematic.
6. User edits profile then immediately sees edit.
7. Usually no: like count can be eventual; seat ownership requires strong correctness.

</details>

## Common Interview Mistakes

- Saying CAP means choose any two permanently.
- Saying eventual means “eventually in milliseconds.”
- Applying one consistency model to the whole product.
- Using stale replica for correctness-sensitive decision.
- Quorum math without version/conflict handling.
- Calling ACID consistency and distributed consistency the same concept.
- Assuming last-write-wins is always safe.
- Choosing global strong consistency without latency/availability cost.

## Must Remember

- **Consistency should be specified per operation/invariant.**
- **Strong consistency costs coordination.**
- **Eventual consistency permits temporary divergence.**
- **Read-after-write is a common user-facing guarantee.**
- **Monotonic reads stop a client from going backward.**
- **CAP trade-off matters during network partitions.**
- **CAP is not “pick any two always.”**
- **Quorum overlap is useful intuition, not the whole consistency story.**
- **Conflict avoidance can be better than conflict resolution.**

## Interview Revision Summary

For each critical operation ask:

```text
Can read be stale?
How stale?
Read own writes?
Monotonic?
Concurrent writers?
Single leader or quorum?
Partition behavior?
Fail closed or stay available?
Conflict strategy?
```

## Explain Without Notes

Explain consistency requirements for:
- user profile update;
- social like count;
- ticket purchase;
- search results.

Then explain CAP using the ticket purchase example.

## Completion Checklist

- [ ] I understand strong/eventual/read-after-write/monotonic consistency.
- [ ] I explain CAP correctly.
- [ ] I understand quorum intuition.
- [ ] I reason about conflicts.
- [ ] I define consistency per operation.
