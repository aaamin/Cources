# Lesson 16 — Partitioning & Sharding

**Phase:** Fundamentals  
**Session:** 16/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn how large datasets and traffic are divided across machines, how shard keys are chosen, and why hotspots and scatter-gather dominate practical design.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Why partition

One database has finite CPU, memory, storage, and I/O. Partitioning gives different machines ownership of different data subsets so capacity can grow horizontally.

## Hash partitioning

Hash the partition key to spread values. Distribution is often even, but natural range locality is lost and range queries may contact many shards.

## Range partitioning

Assign ranges—time, ID, geography—to shards. Efficient range scans are possible, but sequential writes often hotspot the newest range.

## Shard-key choice

A key should distribute load and align with access. `conversation_id` keeps chat history together, but a huge live group can hotspot. `user_id` spreads users but cross-user aggregation becomes harder.

## Hotspots

Balanced data size does not imply balanced traffic. Celebrity accounts, giant tenants, trending items, and current-time buckets can overwhelm one owner.

## Resharding and scatter-gather

Growth may require splitting/moving partitions while serving traffic. Queries that do not contain the shard key can fan to many shards and merge results, creating expensive scatter-gather.

## Worked example — sharding chat messages

Use `hash(conversation_id) → shard` to keep conversation history local and preserve per-conversation ordering. This works until one giant group dominates writes. At that point, split very large conversations into subpartitions while keeping a logical sequence strategy.

## Interview lens

For every proposed shard key, answer two questions: “why is it good?” and “what workload breaks it?”

## What to remember

Sharding is primarily a **key-distribution and access-pattern problem**, not a hashing problem.

## Review after reading

1. Why shard?
2. Hash vs range trade-off?
3. What is a hot partition?
4. Why can conversation_id work?
5. What is scatter-gather?

## Deeper study notes

### Partitioning data and partitioning traffic are related but different

A dataset may be evenly split by row count while one shard receives most traffic. Capacity planning must consider bytes, read QPS, write QPS, and expensive query patterns per partition. A “balanced” key by data size can still be operationally terrible.

### Compound partitioning can solve special hotspots

A single logical key can be split with a suffix: `(tenant_id, bucket)` or `(conversation_id, segment)`. Reads may need to query several buckets and merge results, so this trades write distribution for read complexity. Use it only for pathological large keys/tenants.

### Rebalancing must preserve availability

Moving a partition is not merely copying files. During migration, writes continue. Systems may dual-read, dual-write, stream changes after snapshot copy, or use ownership epochs. At L4, understand that resharding is a live migration problem with correctness and load impact.

### Global secondary access is expensive

If data is partitioned by user but you need queries by email, location, or timestamp, you may build a separate index whose partitioning follows that lookup. That derived index has its own update and consistency behavior.

### Common mistakes

- Choosing timestamp as a write partition key and creating a “latest” hotspot.
- Using random partitioning then expecting efficient global ranges.
- Ignoring giant tenants/groups.
- Assuming resharding is free.
- Failing to distinguish logical ID from physical partition key.


## Important interview ideas

> **Important:** The hardest sharding decision is the **partition key**. A good key distributes not only bytes, but also read/write traffic, while keeping important queries local.

### Partitioning dimensions

A useful shard-key review asks four questions:

```text
1. Is data volume evenly distributed?
2. Is traffic evenly distributed?
3. Are common queries single-shard?
4. Can one tenant/key become enormous?
```

A user-ID hash may distribute users well, but a celebrity user can still create a traffic hotspot. A timestamp range supports time scans but concentrates current writes.

### Hash vs range

Hash partitioning improves distribution but destroys global locality. Range partitioning preserves ordered ranges but risks “latest” hotspots.

You can combine strategies. For example, logs can use `(day, hash(tenant))` so data ages by day while each day's writes spread across buckets.

### Logical vs physical shards

A mature system can define many **logical partitions** and map them to physical machines. Moving a logical shard between machines is easier than rehashing every record. A shard map/control plane tracks ownership.

This is an alternative to consistent hashing and is often easier to reason about for databases.

### Cross-shard operations

Once data is split, operations spanning shards become harder:

- global unique constraints;
- joins;
- totals/aggregations;
- transactions;
- ordered pagination.

Avoid distributing one business invariant across many shards unless necessary.

### Resharding is an online migration

At scale, moving data while writes continue may require:

1. copy snapshot;
2. stream/dual-write changes;
3. verify consistency;
4. switch ownership/version;
5. clean old copy.

At L4, acknowledging this complexity is enough.

## Worked scenario — multi-tenant SaaS

Partition by `tenant_id` keeps each tenant's data local, simplifying queries and isolation. But one enterprise tenant can exceed a shard.

Options:

- dedicate large tenants to their own shards;
- split `(tenant_id, bucket)` for giant tenants;
- use logical shards and move heavy tenants independently.

This illustrates why “even number of tenants” is not enough; tenant sizes and traffic are skewed.

## Interview questions and model answers

### Q1. “How do you choose a shard key?”

Start from access patterns and traffic distribution. I want most reads/writes to include the key, avoid monotonic hotspots, and distribute heavy users. Then I explicitly test pathological tenants/celebrities.

### Q2. “What is a hot partition?”

One partition receives disproportionate load or data. It can happen even with many total shards. Mitigation may require changing key granularity, splitting the hot entity, caching reads, or isolating large tenants.

### Q3. “What happens to cross-shard queries?”

The coordinator may query several/all shards and merge results, increasing latency and load. Often a separate derived index or changed data model is better for a frequent secondary query.

### Q4. “When should I not shard?”

When one database still meets capacity/availability requirements. Sharding adds routing, migration, transaction, operational, and query complexity. Use it for a real limit, not as a default box.

## Common mistakes to avoid

- Timestamp-only current-write shard.
- Partition by tenant with no giant-tenant plan.
- Only checking data balance, not QPS balance.
- Ignoring cross-shard queries.
- Assuming resharding is trivial.
- Sharding before simpler optimizations.

## Short revision note

Shard-key test: **distribution + locality + hotspot risk + cross-shard cost + resharding path**.

## Topics to revise

- [ ] hash partitioning
- [ ] range partitioning
- [ ] geographic partitioning
- [ ] shard-key selection
- [ ] hot partitions
- [ ] scatter-gather
- [ ] logical shards
- [ ] resharding

## Interview-ready synthesis

### A strong 60–90 second explanation

I choose a shard key from both data distribution and traffic distribution. I want important operations to be single-shard, avoid monotonic hot ranges, and explicitly test giant tenants/celebrities. I acknowledge that resharding is an online migration and cross-shard queries/transactions become more expensive.

### How this topic connects to the wider system

- Scalability: partitioning distributes storage and throughput across owners.
- Performance: single-shard locality is faster than scatter-gather.
- Correctness: cross-shard transactions/uniqueness are harder.
- Operations: rebalancing and hot-tenant migration require explicit ownership control.

### Revision flashcards with answers

**Hash sharding benefit?**  
Even distribution for random keys.

**Hash sharding cost?**  
Range/global queries lose locality.

**Range sharding benefit?**  
Efficient ordered/range scans.

**Range sharding risk?**  
Monotonic/current ranges can hotspot.

**What is scatter-gather?**  
Query multiple shards and merge results because partition key is unavailable.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Chat: partition by conversation for history locality, then special-case giant groups.
- Telemetry: partition by tenant/device hash while isolating huge tenants to prevent noisy-neighbor hotspots.
- Ticketing: partition by event/section so authoritative seat contention remains localized.

### When not to overuse this idea

Do not choose a partition key only because it is unique. Uniqueness says nothing about locality, traffic distribution, or cross-shard query cost.

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

**Next:** Lesson 17 — Consistent Hashing
