# Session 15 — Database Replication, Leaders & Failover

## Outcome

You should understand single-leader, multi-leader, and leaderless replication at interview depth; synchronous vs asynchronous replication; replica lag; read-after-write problems; leader failure/failover; split-brain risk; and when different models are useful.

## Why Replicate?

Replication keeps multiple copies of data.

Goals:
- availability;
- durability;
- read scaling;
- geographic proximity;
- failover.

Replication does **not** mean backup. A mistaken delete can replicate to every replica.

## Single-Leader / Leader-Follower

Common model:

```text
          writes
Client ───────→ Leader
                 |
        replication
          /      \
     Follower  Follower
        ↑          ↑
       reads      reads
```

Leader handles writes; followers copy changes and may serve reads.

### Benefits

- clear write authority;
- easier consistency reasoning than multiple writers;
- followers can scale some reads;
- failover is understandable.

### Costs

- leader write bottleneck for one shard;
- leader failure requires promotion;
- follower reads may be stale.

## Synchronous Replication

Leader waits for one/more replica acknowledgements before confirming commit.

Advantages:
- lower risk of losing acknowledged writes if leader fails;
- replica is more current.

Costs:
- higher write latency;
- replica/network slowdown affects availability;
- cross-region sync replication can be expensive in latency.

## Asynchronous Replication

Leader confirms before follower fully catches up.

Advantages:
- lower write latency;
- follower slowness is less directly on write path.

Risks:
- replication lag;
- failover may lose recent acknowledged writes depending on system guarantees;
- stale reads.

## Replica Lag

Example:

```text
T0 User updates profile on leader
T1 Immediate GET routed to follower
T2 Follower still has old profile
```

User sees old data.

This is a **read-after-write** problem.

Mitigations:
- read own recent writes from leader;
- sticky/session routing temporarily;
- version/token to ensure replica caught up;
- synchronous replication;
- tolerate stale data if product allows.

## Read Scaling

Read replicas help when:
- workload is read-heavy;
- stale/lagging reads are acceptable for some paths.

Do not route correctness-sensitive reads blindly to replicas.

Examples:
- analytics/reporting → replica can be fine;
- checking whether last inventory unit is still available → authoritative leader/transaction path.

## Leader Failure

Failure flow conceptually:

```text
Leader fails
   ↓
detect failure
   ↓
choose/promote follower
   ↓
redirect writes
   ↓
old leader eventually returns
```

Hard questions:
- Was old leader truly dead or network-partitioned?
- Which follower is most up to date?
- Could two leaders accept writes?
- What happens to recent unreplicated writes?

## Split Brain

Two nodes both believe they are leader and accept conflicting writes.

Can happen during:
- network partitions;
- poor failover coordination.

Systems use consensus/leases/epochs/fencing internally to prevent or resolve this. At L4, recognize the risk; you do not need to implement Raft/Paxos.

## Leader Election Concept

Nodes/coordination system choose one active leader for a role.

Properties to care about:
- only one valid leader/epoch;
- failure detection;
- promotion;
- stale leader prevented from writing.

Later, fencing tokens deepen this concept.

## Multi-Leader Replication

Multiple leaders accept writes, often in different regions.

```text
Region A Leader ↔ Region B Leader
```

Potential use:
- multi-region local writes;
- occasionally connected clients/sites.

Major cost:
**write conflicts.**

Example:
- user profile updated differently in two regions before replication.

Need conflict strategy:
- last-write-wins (can lose data);
- merge rules;
- application-specific conflict resolution;
- avoid concurrent writes to same entity via home-region ownership.

Multi-leader is not a free active-active solution.

## Leaderless Replication

Clients/coordinators write to multiple replicas without one permanent leader.

Conceptual quorum:

```text
N replicas
write to W
read from R
```

If:
```text
R + W > N
```
there is overlap between read and write quorums, but real consistency still depends on implementation, conflict/version handling, failures, sloppy quorums, etc.

Benefits:
- no single leader;
- high write availability in some designs.

Costs:
- conflict/version resolution;
- read repair;
- stale replicas;
- more complex semantics.

Know recognition-level trade-offs.

## Geographic Replication

Cross-region async replication:
- lower local write latency;
- lag after writes;
- disaster recovery.

Cross-region synchronous replication:
- stronger durability/consistency;
- network RTT on writes;
- region/network issue can reduce availability.

Often choose:
```text
strong within home region
async across regions
```
unless business requires global strong consistency.

## Failover and Data Loss Trade-Off

Suppose:
1. leader commits write to local disk;
2. acknowledges user;
3. has not asynchronously replicated;
4. leader is permanently lost;
5. follower is promoted.

Recent acknowledged write may disappear.

Whether this is possible depends on exact DB durability/replication settings. Interview point: **replication acknowledgement semantics matter.**

## Backup vs Replica

Replica mirrors current state.

If someone executes:
```sql
DELETE FROM users;
```
that delete replicates.

Backup:
- historical recovery point;
- helps recover from corruption/deletion/ransomware-like logical errors.

You need both for different failure classes.

## Worked Example — User Profile

Requirements:
- profile reads huge;
- writes rare;
- user should see own edit immediately;
- other users may see update seconds later.

Design:
- single leader;
- async read replicas;
- normal reads from replicas;
- after update, route that user's read to leader or use consistency token;
- replication lag monitored.

This gives read scale without requiring all reads to be strongly current.

## Small Design Drills

1. Why can async followers serve stale data?
2. Why can synchronous cross-region replication hurt latency?
3. What is split brain?
4. Why is a replica not a backup?
5. Multi-leader reduces remote-write latency but introduces what major problem?
6. What user-facing anomaly appears when write goes to leader and immediate read goes to lagging follower?

<details>
<summary>Answer key</summary>

1. Leader acknowledges before follower has applied the write.
2. Commit waits for network/replica acknowledgements across long distance.
3. Multiple nodes believe they are leader and accept conflicting writes.
4. Logical corruption/deletes are replicated; backup provides historical recovery.
5. Concurrent write conflicts and resolution.
6. Read-after-write violation/stale read.

</details>

## Common Interview Mistakes

- “Read replicas make reads strongly consistent.”
- “Replica = backup.”
- Ignoring promotion/failover time.
- Saying active-active without conflict strategy.
- Assuming synchronous replication is always best.
- Ignoring stale reads after user writes.
- Going too deep into consensus implementation for a normal L4 prompt.
- Using read replicas for correctness-critical decisions without justification.

## Must Remember

- **Replication serves availability/durability/read scale, but creates consistency questions.**
- **Async replication permits lag.**
- **Sync replication adds latency and availability coupling.**
- **Read-after-write may fail on lagging replicas.**
- **Leader failover risks split brain/stale leaders.**
- **Multi-leader requires conflict handling.**
- **Leaderless systems use replica/quorum/conflict semantics.**
- **A replica is not a backup.**
- **State the replication model, not just “we have replicas.”**

## Interview Revision Summary

For replication ask:

```text
Who accepts writes?
Sync or async?
Who serves reads?
Allowed staleness?
Read-after-write needed?
Leader failure?
Promotion?
Recent write loss?
Split brain?
Cross-region?
Backup separately?
```

## Explain Without Notes

Design replication for a global profile service where local reads should be fast, users must see their own edits immediately, but other users may see them a few seconds late.

## Completion Checklist

- [ ] I understand single-leader replication.
- [ ] I compare sync vs async.
- [ ] I understand lag/read-after-write.
- [ ] I recognize multi-leader conflicts.
- [ ] I recognize leaderless/quorum models.
- [ ] I understand failover/split brain.
- [ ] I distinguish replicas from backups.
