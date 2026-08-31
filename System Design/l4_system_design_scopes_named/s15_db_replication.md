# Session 15 — Database Replication, Leaders & Failover

## 1. Must Learn

### Leader-follower replication
- **Understand:** Understand writes to a leader and replicated copies used for durability/read scaling.
- **Decision/trade-off:** Write simplicity/consistency vs leader dependency and replica lag.

### Sync vs async replication
- **Understand:** Know synchronous replication waits for replica confirmation; asynchronous does not.
- **Decision/trade-off:** Durability/freshness vs write latency/availability.

### Read replicas & lag
- **Understand:** Understand replicas can serve stale reads and cause read-after-write surprises.
- **Decision/trade-off:** Read scale/latency vs freshness.

### Leader failure & failover
- **Understand:** Know detection, replica promotion, routing update, and recovery conceptually.
- **Decision/trade-off:** Availability/recovery speed vs risk of data loss/split brain.

### Replication conflicts
- **Understand:** Understand conflicts can arise in multi-writer/leaderless designs and require resolution.
- **Decision/trade-off:** Write availability/geographic locality vs conflict complexity.

### Split brain
- **Understand:** Recognize danger of multiple nodes believing they are leader/writable owner.
- **Decision/trade-off:** Availability during uncertainty vs correctness.

## 2. Should Know

- Multi-leader and leaderless models at comparison/recognition depth.
- Promotion should consider replica freshness.
- Users may need read-your-write behavior after a write.

## 3. Recognition Only

- Consensus/election algorithm internals
- Replication log formats

## 4. Important Comparisons

- Synchronous vs asynchronous replication.
- Leader-follower vs multi-leader vs leaderless.
- Leader read vs replica read.
- Availability during failover vs strict correctness/data-loss risk.

## 5. Important Interview Questions

1. What can users observe when reading from a lagging replica?
2. When should a read go to the leader instead?
3. What happens if the leader fails before an async replica receives the latest write?
4. How is a new leader chosen conceptually?
5. What is split brain and why is it dangerous?
6. When are multi-leader/leaderless models worth their conflict complexity?

## 6. Common Interview Mistakes

- **Assuming replicas are instantly current** → Discuss replication lag.
- **Using read replicas for consistency-sensitive reads blindly** → Route or coordinate when read-after-write matters.
- **Saying failover is free** → There is detection/promotion/routing delay and possible data loss.
- **Ignoring split brain** → Protect single-writer ownership when required.
- **Deep-diving consensus algorithms** → L4 needs failover behavior/trade-offs, not algorithm implementation.

## 7. Communication

### Important Vocabulary

leader, follower, replica, read replica, synchronous replication, asynchronous replication, replication lag, failover, promotion, stale read, read-after-write, split brain, conflict

### Useful Interview Phrases

- “Async replication improves write latency but creates a window where replicas are stale.”
- “For read-after-write, I may route that user's read to the leader or an adequately caught-up replica.”
- “Failover restores availability, but I need to discuss detection time and possible lost writes.”

### Important Questions to Ask the Interviewer

- **Question:** “Can users tolerate stale reads?”  
  **Why it matters:** Determines replica-read strategy.
- **Question:** “How much write latency is acceptable?”  
  **Why it matters:** Affects sync vs async replication.
- **Question:** “What data-loss window is acceptable during failover?”  
  **Why it matters:** Changes replication/DR choices.

## 8. ⭐ Must Remember

1. Replication improves redundancy/read capacity but creates freshness/failover questions.
2. Async replicas can lag.
3. Read replicas can violate read-after-write expectations.
4. Sync replication costs write latency/availability.
5. Failover has a detection and promotion window.
6. Split brain threatens correctness.

## 9. Study Priority

1. Study first: leader-follower, read replicas, sync/async.
2. Study next: lag and read-after-write behavior.
3. Finish with: leader failure, promotion, split brain, other leader models recognition.

## 10. Revision Checklist

- [ ] Explain leader-follower replication.
- [ ] Compare sync vs async.
- [ ] Explain stale/read-after-write behavior.
- [ ] Walk through leader failover.
- [ ] Recognize split brain and multi-leader/leaderless trade-offs.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
