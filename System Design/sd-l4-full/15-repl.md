# Lesson 15 — Database Replication & Failover

**Phase:** Fundamentals  
**Session:** 15/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn leader/follower replication, synchronous vs asynchronous copies, replication lag, read replicas, failover, and split-brain risk.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Leader/follower

A leader accepts writes and followers copy changes. Followers can add read capacity and redundancy. On leader failure, a follower may be promoted.

## Sync vs async

Synchronous replication waits for replicas before acknowledging, increasing durability/freshness but adding latency and potentially reducing availability. Asynchronous replication acknowledges earlier but can lose recent unreplicated writes on leader failure.

## Replication lag

Followers may be behind. A user can write a profile update then read old data from a follower. This read-after-write anomaly can be handled by leader reads for recent writers, version-aware routing, or accepting temporary staleness.

## Read replicas

Followers are useful for read-heavy traffic, but in a standard single-leader design they do not increase write throughput. They also create freshness considerations.

## Failover

Failover must detect leader loss, elect/promote a replacement, and route clients to it. False detection and old-leader return can create conflicting leadership.

## Split brain

Split brain is when multiple nodes believe they are leader and accept writes. Quorums, consensus, fencing, and careful failover mechanisms prevent it. L4 candidates need to understand the risk, not implement consensus.

## Worked example — profile update with replica lag

The leader commits a new display name, but a follower has not replayed it. If the immediate GET goes to that follower, the old name appears. The product can route that user's immediate reads to the leader or accept bounded staleness for non-critical fields.

## Interview lens

Whenever you add replicas, mention lag. Whenever you discuss failover, mention promotion, client rerouting, and potential loss of unreplicated writes.

## What to remember

Replication improves availability/read capacity but introduces lag, leader failover, and consistency questions.

## Review after reading

1. Leader role?
2. Async replication data-loss risk?
3. Replication lag?
4. Why no write scaling from read replicas?
5. What is split brain?

## Deeper study notes

### Replication has at least three goals

Replication can improve durability, availability, and read throughput. These goals are related but not identical. A follower used for disaster recovery may be far away and unsuitable for low-latency reads. A local read replica may improve reads but not protect from a regional disaster.

### Acknowledgement defines data-loss exposure

If the leader acknowledges before any replica has the write, a leader crash may lose that acknowledged change. Waiting for multiple replicas lowers that risk but increases latency and can make writes unavailable when replicas are unreachable. This is a classic durability/latency/availability trade-off.

### Failover has a recovery timeline

Think through detection → promotion → client discovery → traffic restoration → repair of the old leader. Even if promotion is automatic, applications may see errors during the transition. Connection pools may need to reconnect and cached topology may be stale.

### Replica reads need a policy

You can route analytics or stale-tolerant product reads to followers while keeping critical read-after-write paths on the leader. This is often better than a blanket “all reads go to replicas.” Routing can be based on operation semantics.

### Common mistakes

- Saying replication automatically gives strong consistency.
- Forgetting asynchronous data-loss windows.
- Sending immediate post-write reads to a lagging replica without considering UX.
- Assuming failover is instantaneous.
- Placing all replicas in one failure domain.


## Important interview ideas

> **Important:** Replication gives copies; it does not automatically give strong consistency. The acknowledgement policy and read routing determine what users can observe.

### Replication goals

Separate three goals:

1. **availability** — another copy can serve after failure;
2. **durability** — data survives machine/zone loss;
3. **read capacity** — replicas answer reads.

One topology may optimize one goal more than another. A far-away disaster-recovery replica improves regional survivability but may be too stale/slow for user reads.

### Write acknowledgement

If the leader acknowledges after only local persistence, latest writes may be lost if the leader dies before replication. Waiting for one or more synchronous replicas reduces that window but adds latency and can make writes fail when replicas are unreachable.

This is a direct **latency/availability/durability** trade-off.

### Read routing policy

Do not say “all reads go to replicas.” Ask which reads tolerate lag.

- analytics/reporting → replicas are usually fine;
- public product details → small lag may be fine;
- immediately reading your saved settings → route to leader or enforce session consistency;
- inventory decision → stale read may be dangerous.

### Failover steps

Conceptually:

```text
detect leader failure
→ choose/promote follower
→ prevent old leader from accepting writes
→ update routing
→ reconnect clients
→ repair/rejoin old leader
```

“Automatic failover” still has a recovery interval and ambiguous in-flight operations.

## Worked scenario — async replica loss window

Write W1 is committed on leader and acknowledged. Before W1 reaches follower, leader's disk fails permanently. Follower becomes leader. W1 is gone even though client saw success.

If this is unacceptable, use synchronous replication/quorum for that data or another durability mechanism. If losing a few seconds of analytics events is acceptable, asynchronous replication may be fine.

## Interview questions and model answers

### Q1. “Why use read replicas?”

To increase read throughput and isolate read-heavy workloads from the writer. I would use them only for reads that can tolerate their replication lag or route session-critical reads to the leader.

### Q2. “What happens during failover?”

There is detection and promotion time. Some requests fail or time out. Clients reconnect to the new leader. With async replication, latest acknowledged writes may be missing. The old leader must be fenced before rejoining to avoid split brain.

### Q3. “Synchronous or asynchronous replication?”

Synchronous gives stronger durability/freshness but increases write latency and may reduce availability. Async improves latency/availability but creates lag and data-loss windows. Choose per business requirement.

### Q4. “Can replicas be in the same zone?”

They can, but that does not protect from zone failure. For high availability, copies should span independent failure domains; for disaster recovery, possibly regions.

## Common mistakes to avoid

- Replication = strong consistency.
- Read replica = write scaling.
- No discussion of lag.
- Failover assumed instant.
- All replicas in same failure domain.
- Ignoring old leader/split-brain fencing.

## Short revision note

Replication questions: **where are copies, when is a write acknowledged, where do reads go, what lag is acceptable, and what exactly happens at failover?**

## Topics to revise

- [ ] leader/follower
- [ ] sync vs async
- [ ] replication lag
- [ ] read-after-write
- [ ] read replicas
- [ ] failover timeline
- [ ] split brain/fencing
- [ ] failure domains

## Interview-ready synthesis

### A strong 60–90 second explanation

I explain replication by write acknowledgement and read routing. Copies can improve availability, durability, and read throughput, but asynchronous followers lag and failover has a recovery interval. I specify which reads can be stale and which writes need synchronous durability.

### How this topic connects to the wider system

- Reliability: replicas in independent failure domains allow failover.
- Correctness: lag creates read-after-write anomalies and possible data-loss windows.
- Performance: read replicas offload stale-tolerant queries.
- Global architecture: cross-region replicas improve DR but add latency/freshness trade-offs.

### Revision flashcards with answers

**Async replication risk?**  
Leader can fail before acknowledged write reaches follower, losing recent data.

**Read replica scales what?**  
Reads, not primary write throughput.

**Split brain?**  
Multiple nodes believe they can accept authoritative writes, causing conflicts.

**Failover steps?**  
Detect, promote/fence, update routing, reconnect, repair old node.

**When route read to leader?**  
When immediate read-after-write or current authoritative value is required.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Profile service: stale follower reads after update show why read-after-write routing can matter.
- News catalog: stale-tolerant reads can move to replicas and reduce leader load.
- Disaster recovery: a cross-region replica protects region loss but has a different latency/lag role from a local read replica.

### When not to overuse this idea

Do not advertise a replica as backup against accidental deletion/corruption; replication can copy the mistake immediately.

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

**Next:** Lesson 16 — Partitioning & Sharding
