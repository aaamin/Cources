# Session 16 — Partitioning & Sharding

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn how a dataset is split across machines and how partition-key choice affects scale, queries, and hotspots.

## What You Need to Read / Learn

- Horizontal partitioning/sharding.
- Hash partitioning for distribution.
- Range partitioning for ordered/range access.
- Geographic partitioning.
- Partition key selection.
- Hot partitions from skew, time, or celebrity users.
- Scatter-gather queries across shards.
- Rebalancing and resharding.
- Cross-shard transactions/joins and why they are harder.
- Directory/routing metadata conceptually.

## What You Need to Do

- [ ] Choose shard keys for users, messages, rides, posts, and telemetry.
- [ ] For every choice, invent an adversarial workload that breaks it.
- [ ] Explain how a time-based key can create a hot newest partition.

## **Must Remember for the Interview**

- **The partition key is one of the most important scalability decisions in a distributed datastore.**
- **Evenly distributed data does not guarantee evenly distributed traffic.**
- **Queries that do not include the partition key may require fan-out/scatter-gather.**
- **Resharding is an operational cost; plan for growth.**
- **Hotspots usually come from workload skew, not average data size.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Hash partition → even distribution; range partition → efficient ranges but hotspot risk.**
- **Choose keys from dominant access patterns + traffic distribution.**
- **Avoid cross-shard operations in the critical path where possible.**
- **Ask 'What happens for a celebrity/hot tenant/current time bucket?'**
- **State how routing and rebalancing work conceptually.**

## Self-Test Before Marking This Session Complete

- [ ] Can I choose and defend a shard key?
- [ ] Can I explain scatter-gather?
- [ ] Can I give a hot-partition example?
- [ ] Can I compare hash vs range partitioning?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 16/46  
**Next:** Session 17
