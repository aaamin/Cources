# Session 16 — Partitioning, Sharding & Distributed IDs

## Outcome

You should be able to choose and critique shard keys, compare hash/range/geo/time partitioning, recognize hot partitions and scatter-gather queries, explain rebalancing/resharding, and select a distributed ID strategy based on uniqueness, ordering, decentralization, unpredictability, and shard behavior.

## Partitioning vs Sharding

Terminology varies, but for interview purposes:

- **partitioning**: splitting data into subsets;
- **sharding**: distributing those partitions across nodes/databases.

Example:

```text
Users A–F → Shard 1
Users G–M → Shard 2
Users N–Z → Shard 3
```

Goal: one node no longer carries all storage/write/read load.

## When to Shard

Do not shard by default.

First ask:
- storage exceeds one node?
- write throughput exceeds cluster?
- read replicas/cache insufficient?
- operational/availability boundary needed?
- tenant isolation?

Sharding adds:
- routing;
- rebalancing;
- cross-shard queries;
- transactions;
- secondary index complexity;
- hot partitions.

## Hash Partitioning

```text
shard = hash(key) mod N
```

Advantages:
- usually even distribution if key is well distributed;
- good point lookup.

Weaknesses:
- range scans across key order are poor;
- changing N with naive modulo moves much data;
- one hot key is still hot.

Consistent hashing can reduce movement in some systems.

## Range Partitioning

Example:

```text
A–F
G–M
N–Z
```

or:
```text
timestamps Jan–Mar
Apr–Jun
...
```

Advantages:
- ordered/range queries remain local.

Risks:
- sequential writes can target newest range;
- uneven ranges;
- hotspot at tail.

## Geographic Partitioning

Place data by region/home geography.

Advantages:
- low local latency;
- data residency;
- region-local operations.

Costs:
- users move/travel;
- cross-region interactions;
- failover ownership;
- global queries.

## Time-Based Partitioning

Useful for:
- logs;
- metrics;
- events.

Example:
```text
metrics_2026_08_31
```

Benefits:
- retention/delete old partitions;
- time-range query pruning.

Risk:
- current partition gets all writes.

Combine time with hash/tenant/device bucket:

```text
(day, hash(device_id)%K)
```

## Choosing a Partition Key

Good shard key should generally:
- distribute load/storage;
- support common access patterns;
- keep related transactional data together where possible;
- avoid unbounded hot entities.

Example chat:
```text
conversation_id
```
keeps messages together.

But a giant group chat can become hot.

Mitigation:
```text
(conversation_id, time_bucket)
```
or sequence bucket, while accepting multi-bucket reads.

## Hot Partitions

A partition becomes disproportionately busy.

Causes:
- celebrity user;
- one tenant;
- current timestamp range;
- monotonically increasing key;
- poor hash/input distribution.

Solutions:
- key salting/bucketing;
- split hot tenant;
- cache;
- replicate read path;
- isolate heavy tenant;
- change partition strategy.

But salting often makes reads more complex because query must fan out across buckets.

## Scatter-Gather

Query does not contain shard key:

```text
find all users with email X
```

when sharded by `user_id`.

Router must:
- query every shard;
- or use a global index/directory.

Scatter-gather increases latency/cost and becomes harder as shard count grows.

## Resharding / Rebalancing

As data grows:
- add shards;
- move partitions;
- update routing.

Challenges:
- data movement;
- serving reads/writes during migration;
- dual routing;
- avoiding lost/duplicate writes;
- hot range split.

A directory-based mapping can make movement explicit:

```text
tenant123 → shard7
```

but directory itself becomes important infrastructure.

## Distributed ID Requirements

Do not choose UUID blindly.

Ask:
- globally unique?
- unique only per parent/shard?
- sortable?
- roughly time-ordered?
- unpredictable/public?
- generated without central DB?
- compact?
- high write rate?
- does key distribution affect storage hot spots?

## Auto-Increment / Database Sequence

Example:
```text
1, 2, 3, 4...
```

Advantages:
- simple;
- compact;
- ordered.

Problems:
- centralized generation;
- multi-region/shard coordination;
- predictable;
- sequential insert hotspot in some storage layouts.

Works perfectly well for many systems.

## Central ID Service

Dedicated service allocates unique IDs/ranges.

Can generate:
- monotonic sequences;
- blocks per node.

Advantages:
- controlled format/order.

Costs:
- availability/coordination;
- service dependency.

Range allocation reduces per-ID round trips.

## UUID

Large random-ish identifier space.

Advantages:
- decentralized generation;
- extremely low collision probability;
- useful across services/regions.

Trade-offs:
- larger than integer;
- random UUID ordering can hurt locality/index storage;
- not naturally chronological;
- user-facing URLs are long.

Time-ordered UUID variants exist, but at L4 recognition depth simply understand random-vs-ordered trade-off.

## Snowflake-Style IDs

Conceptual structure:

```text
timestamp | node/worker id | sequence
```

Properties:
- unique across workers if configuration is correct;
- roughly time-sortable;
- generated without central request for every ID.

Challenges:
- worker ID assignment;
- clock movement/skew;
- timestamp dependence;
- IDs reveal rough creation time;
- ordering not necessarily perfectly global.

You do not need exact bit counts unless interviewer asks.

## Random Short IDs

URL shorteners may generate random base62 aliases.

Need:
- collision detection/retry;
- enough entropy;
- reserved/unsafe alias rules.

If using deterministic sequence then base62 encoding:
- no collision if sequence unique;
- predictable unless obfuscated.

## IDs and Hot Storage

Monotonically increasing IDs can direct all new writes to one range partition/page/shard depending on datastore/index layout.

Random IDs distribute writes but lose locality.

Time-sortable IDs compromise:
- good chronological locality;
- may still concentrate recent writes.

Storage engine specifics vary. State the trade-off rather than universal rule.

## Worked Example — Chat Messages

Requirements:
- unique message IDs;
- ordered within conversation;
- offline replay;
- huge conversation history.

Possible model:
```text
partition: conversation_id + bucket
sort: sequence
message_id: globally unique time-sortable ID
```

Ordering should rely on explicit per-conversation sequence/order, not assume globally generated IDs perfectly represent delivery order.

## Small Design Drills

1. Why can `created_at` alone be a terrible shard key for a write-heavy stream?
2. What is scatter-gather?
3. Why can user_id be good for user-centric data?
4. When is auto-increment perfectly acceptable?
5. What does Snowflake-style ID combine?
6. Why might random UUIDs hurt some index locality?
7. Does consistent hashing fix a hot celebrity key?

<details>
<summary>Answer key</summary>

1. Current time range receives all writes.
2. Querying many/all shards because query lacks routing key.
3. User's data/operations stay colocated and hash(user_id) often distributes users.
4. Single-region/DB or when central sequence is simple and its limits are acceptable.
5. Time + worker/node identity + local sequence.
6. Inserts arrive across the keyspace rather than append locality, increasing storage/index overhead in some systems.
7. No. One hot key still maps to limited ownership unless separately handled/replicated.

</details>

## Common Interview Mistakes

- Sharding before one node/cluster needs it.
- Choosing timestamp-only range partition for high writes.
- Saying “hash makes everything balanced” while one key is hot.
- Ignoring scatter-gather.
- Ignoring rebalancing.
- Choosing UUID because “distributed system.”
- Assuming Snowflake gives perfect global ordering.
- Mixing entity ID and business ordering guarantees.

## Must Remember

- **Shard only when scale/ownership justifies the complexity.**
- **Partition key determines distribution and query locality.**
- **Hash distributes keys; range preserves order but can hotspot.**
- **One hot key remains hot even with many shards.**
- **Queries without shard key may scatter-gather.**
- **Resharding must preserve correctness during movement.**
- **ID strategy follows uniqueness/order/decentralization needs.**
- **Auto-increment is not inherently wrong.**
- **Snowflake-style IDs are time + worker + sequence conceptually.**
- **Do not use ID order as business ordering unless guaranteed.**

## Interview Revision Summary

Shard-key checklist:
```text
Cardinality?
Distribution?
Hot entities?
Common reads?
Common writes?
Transactions local?
Range query?
Secondary lookup?
Growth?
Rebalance?
```

ID checklist:
```text
Unique scope?
Ordered?
Public/predictable?
Central generator allowed?
Multi-region?
Compact?
Clock dependence?
Storage locality?
```

## Explain Without Notes

Design partitioning and ID generation for a messaging service with billions of messages, mostly small conversations, but some groups have millions of members.

## Completion Checklist

- [ ] I can compare hash/range/geo/time partitioning.
- [ ] I identify hot partitions.
- [ ] I understand scatter-gather.
- [ ] I understand resharding concerns.
- [ ] I can compare sequence/UUID/Snowflake/random IDs.
- [ ] I separate ID uniqueness from ordering semantics.
