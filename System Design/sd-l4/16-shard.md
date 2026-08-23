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
