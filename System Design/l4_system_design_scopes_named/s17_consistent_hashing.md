# Session 17 — Consistent Hashing

## 1. Must Learn

### Hash-ring intuition
- **Understand:** Understand mapping nodes and keys to a ring so ownership changes locally when nodes join/leave.
- **Decision/trade-off:** Reduced movement vs implementation/operational complexity.

### Node addition/removal
- **Understand:** Know only a portion of keys should move compared with naive modulo hashing.
- **Decision/trade-off:** Rebalancing efficiency vs transient load during movement.

### Virtual nodes
- **Understand:** Understand multiple logical positions per physical node improve balance/flexibility.
- **Decision/trade-off:** Better distribution vs metadata/management overhead.

### Partition ownership
- **Understand:** Know how a client/router determines the node responsible for a key.
- **Decision/trade-off:** Deterministic routing vs topology-awareness needs.

### Uneven distribution & skew
- **Understand:** Consistent hashing balances key-space ownership, not necessarily request popularity.
- **Decision/trade-off:** Balanced partitions vs hot keys/workload skew.

### Replication around ring concept
- **Understand:** Recognize adjacent/other ring positions can hold replicas.
- **Decision/trade-off:** Availability vs consistency/storage complexity.

## 2. Should Know

- Use cases: distributed caches and partitioned storage.
- Compare with modulo hashing during cluster resize.
- Recognize topology/heterogeneous-node weighting concerns.

## 3. Recognition Only

- Rendezvous hashing
- Ring gossip/member-discovery internals

## 4. Important Comparisons

- Modulo hashing vs consistent hashing.
- Physical nodes vs virtual nodes.
- Balanced key distribution vs balanced traffic.
- Partition ownership vs replication.

## 5. Important Interview Questions

1. Why does modulo hashing cause large remapping when node count changes?
2. How does consistent hashing reduce data movement?
3. Why are virtual nodes useful?
4. What happens when a node is removed?
5. Why doesn't consistent hashing solve hot keys?
6. What consistency problems remain after replication?

## 6. Common Interview Mistakes

- **Claiming no data movement** → It reduces movement; it does not eliminate it.
- **Claiming it solves hot keys** → Popularity skew is separate from key-space balance.
- **Ignoring virtual nodes** → Know their balancing purpose.
- **Treating ring replication as consistency solution** → Replication consistency still needs separate design.
- **Using it when topology is static/simple** → Only add when node churn/repartitioning benefit matters.

## 7. Communication

### Important Vocabulary

consistent hashing, hash ring, virtual node, partition ownership, rebalancing, node churn, replication, skew, hot key

### Useful Interview Phrases

- “The main benefit is limiting remapping when the node set changes.”
- “Virtual nodes make ownership more evenly distributed.”
- “This balances key-space ownership, but a single popular key can still be hot.”

### Important Questions to Ask the Interviewer

- **Question:** “Will nodes be added/removed frequently?”  
  **Why it matters:** Determines value of consistent hashing.
- **Question:** “Is traffic popularity skewed?”  
  **Why it matters:** Determines whether hot-key mitigation is also needed.
- **Question:** “Do we need replicated ownership?”  
  **Why it matters:** Adds availability/consistency considerations.

## 8. ⭐ Must Remember

1. Consistent hashing reduces remapping during membership changes.
2. Virtual nodes improve balance.
3. It does not eliminate rebalancing.
4. It does not solve hot keys.
5. It does not solve replication consistency.
6. Common use: distributed caches/partition ownership.

## 9. Study Priority

1. Study first: naive remapping problem and hash-ring intuition.
2. Study next: node changes and virtual nodes.
3. Finish with: skew, replication, and limits.

## 10. Revision Checklist

- [ ] Explain why modulo hashing remaps heavily.
- [ ] Explain the ring and key ownership.
- [ ] Explain virtual nodes.
- [ ] Walk through node addition/removal.
- [ ] State clearly what consistent hashing does not solve.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
