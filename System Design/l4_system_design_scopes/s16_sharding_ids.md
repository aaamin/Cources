# Session 16 — Partitioning, Sharding & Distributed IDs

## 1. Must Learn

### Why partition/shard
- **Understand:** Understand splitting data/work across nodes when one node cannot handle capacity/throughput.
- **Decision/trade-off:** Horizontal scale vs cross-partition complexity.

### Partition-key choice
- **Understand:** Choose a key that spreads load/data and aligns with access patterns.
- **Decision/trade-off:** Distribution/locality vs hot partitions/scatter-gather.

### Hash vs range partitioning
- **Understand:** Hash spreads keys; range preserves locality/order but can create hotspots.
- **Decision/trade-off:** Uniform distribution vs range/locality queries.

### Hot partitions & skew
- **Understand:** Understand uneven tenants/keys/time ranges can dominate a shard.
- **Decision/trade-off:** Simple keying vs skew mitigation/special handling.

### Scatter-gather
- **Understand:** Know queries without partition-key locality may fan out to many shards.
- **Decision/trade-off:** Flexible queries vs latency/cost.

### Rebalancing/resharding
- **Understand:** Understand data moves as capacity/topology changes.
- **Decision/trade-off:** Operational complexity vs long-term scalability.

### Distributed ID requirements
- **Understand:** Choose IDs based on uniqueness, ordering, decentralization, unpredictability, collision risk, and shard distribution.
- **Decision/trade-off:** Simple centralized sequences vs decentralized/time-sortable/random IDs.

## 2. Should Know

- Geographic, tenant, and time-based partitioning as useful variants.
- Relationship between ID ordering and shard hot spots.
- Central ID service vs UUID vs Snowflake-style IDs conceptually.

## 3. Recognition Only

- Consistent hashing belongs mainly to Session 17
- Database-specific resharding internals

## 4. Important Comparisons

- Hash vs range partitioning.
- Tenant vs geographic vs time-based partitioning.
- Central sequence vs UUID/random vs Snowflake-style/time-sortable ID.
- Locality-friendly key vs evenly distributed key.
- Single-shard query vs scatter-gather query.

## 5. Important Interview Questions

1. What workload breaks this shard key?
2. Will any tenant/key/time range become hot?
3. Can important queries route to one shard?
4. What happens when we need to add shards?
5. What properties must IDs have: uniqueness, ordering, unpredictability, decentralization?
6. Could the ID strategy itself create a hot shard?

## 6. Common Interview Mistakes

- **Choosing user_id/shard key automatically** → Test it against skew and query patterns.
- **Ignoring hot tenants** → Plan for whales/celebrities/skew.
- **Range-sharding monotonically increasing data blindly** → Recognize newest-range hot spots.
- **Cross-shard queries everywhere** → Prefer partition-local access where possible.
- **Choosing IDs only for uniqueness** → Consider ordering, coordination, privacy, and distribution.

## 7. Communication

### Important Vocabulary

partition, shard, partition key, hash partitioning, range partitioning, hot partition, scatter-gather, rebalancing, resharding, UUID, sequence, Snowflake-style ID

### Useful Interview Phrases

- “The shard key should both distribute load and support the dominant access pattern.”
- “The failure mode of this key is a hot tenant.”
- “I’d avoid a design that requires fan-out to every shard for the common query.”

### Important Questions to Ask the Interviewer

- **Question:** “Is traffic evenly distributed across users/tenants?”  
  **Why it matters:** Determines hot-shard risk.
- **Question:** “Which queries must be single-digit-millisecond?”  
  **Why it matters:** Pushes toward shard-local access.
- **Question:** “Do IDs need ordering or unpredictability?”  
  **Why it matters:** Determines ID strategy.

## 8. ⭐ Must Remember

1. Shard keys are workload decisions.
2. Always test shard keys against skew.
3. Hash favors distribution; range favors locality/order.
4. Cross-shard queries cost more.
5. Resharding is operationally significant.
6. ID design can affect sharding and hotspots.

## 9. Study Priority

1. Study first: partitioning purpose, shard key, hash vs range.
2. Study next: hot partitions, scatter-gather, rebalancing.
3. Finish with: ID strategies and shard interaction.

## 10. Revision Checklist

- [ ] Choose a shard key and name its failure mode.
- [ ] Compare hash and range partitioning.
- [ ] Explain scatter-gather cost.
- [ ] Describe resharding conceptually.
- [ ] Choose an ID strategy from requirements.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
