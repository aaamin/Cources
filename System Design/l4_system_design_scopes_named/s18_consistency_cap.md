# Session 18 — Consistency, CAP & Quorums

## 1. Must Learn

### Consistency requirements per operation
- **Understand:** Distinguish strong, eventual, and read-after-write needs; define consistency for important operations/invariants.
- **Decision/trade-off:** Freshness/correctness vs latency/availability.

### Strong vs eventual consistency
- **Understand:** Know strong reads reflect latest agreed state; eventual replicas may temporarily diverge.
- **Decision/trade-off:** Correctness/freshness vs availability/latency/coordination.

### Read-after-write consistency
- **Understand:** Understand a user's post-write read should see their own update when required.
- **Decision/trade-off:** User experience/correctness vs replica-routing/coordination cost.

### CAP during partitions
- **Understand:** Understand that when a network partition prevents communication, a distributed system must trade availability against strong consistency for affected operations.
- **Decision/trade-off:** Continue serving vs reject/delay operations to preserve consistency.

### Read/write quorums
- **Understand:** Understand quorum intuition: require enough replica responses/acks to overlap reads and writes under assumptions.
- **Decision/trade-off:** Stronger freshness/durability vs latency/availability.

### Stale reads & conflicts
- **Understand:** Know replicas can diverge and multi-writer systems may need conflict detection/resolution.
- **Decision/trade-off:** Write availability vs reconciliation complexity.

## 2. Should Know

- Monotonic reads and session consistency conceptually.
- Quorum formulas are less important than overlap intuition and failure behavior.
- Consistency is often different for different operations in the same system.

## 3. Recognition Only

- Vector clocks
- CRDT internals
- Paxos/Raft internals

## 4. Important Comparisons

- Strong vs eventual consistency.
- Read-after-write vs generic eventual consistency.
- Consistency vs availability during a network partition.
- Read quorum vs write quorum.
- Global system-level label vs per-operation consistency requirement.

## 5. Important Interview Questions

1. What consistency does this operation actually require?
2. What user-visible behavior is acceptable from a stale replica?
3. What happens during a network partition?
4. When should the system reject writes rather than risk conflicting state?
5. How do read/write quorums change latency and availability?
6. How are replication conflicts handled?

## 6. Common Interview Mistakes

- **“The whole system is strongly consistent”** → Specify consistency per operation/invariant.
- **Explaining CAP as a normal latency choice** → CAP's key trade-off appears under network partition.
- **Using eventual consistency for reservations/payments casually** → Start from correctness invariant.
- **Memorizing quorum math without failure reasoning** → Explain overlap, latency, and unavailable-replica behavior.
- **Confusing ACID consistency with distributed consistency** → Keep the concepts separate.

## 7. Communication

### Important Vocabulary

strong consistency, eventual consistency, read-after-write, monotonic read, session consistency, CAP, network partition, read quorum, write quorum, stale read, conflict resolution

### Useful Interview Phrases

- “I’d define consistency per operation rather than label the whole system.”
- “During a partition, preserving this invariant may require rejecting or delaying some operations.”
- “Read-after-write matters for the user's own update even if other reads may be eventual.”

### Important Questions to Ask the Interviewer

- **Question:** “Can this operation temporarily return stale data?”  
  **Why it matters:** Determines consistency level.
- **Question:** “What is worse during a partition: unavailability or conflicting state?”  
  **Why it matters:** Clarifies CAP-side behavior.
- **Question:** “Must users immediately see their own writes?”  
  **Why it matters:** Determines read-after-write requirement.

## 8. ⭐ Must Remember

1. Consistency is per operation/invariant.
2. Strong and eventual consistency trade coordination for availability/latency differently.
3. Read-after-write is often a useful middle requirement.
4. CAP matters specifically during network partitions.
5. Quorums trade latency/availability for replica overlap.
6. Stale reads/conflicts need explicit handling.

## 9. Study Priority

1. Study first: strong/eventual/read-after-write.
2. Study next: CAP under partitions.
3. Finish with: quorum intuition, stale reads, conflict handling.

## 10. Revision Checklist

- [ ] Assign consistency requirements to several operations.
- [ ] Explain CAP correctly.
- [ ] Explain read/write quorum intuition.
- [ ] Discuss stale reads and conflicts.
- [ ] Avoid system-wide vague consistency claims.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
