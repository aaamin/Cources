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
