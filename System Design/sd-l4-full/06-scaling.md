# Lesson 06 — Horizontal & Vertical Scaling

**Phase:** Fundamentals  
**Session:** 6/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn scale-up vs scale-out, why statelessness matters, how bottlenecks move, and why capacity changes should follow measured pressure.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Vertical scaling

Scale up by adding CPU, memory, faster storage, or network to one machine. It is operationally simple and often the best early choice. Limits include hardware ceilings, high-end cost, and larger failure blast radius.

## Horizontal scaling

Scale out by adding machines. It can improve capacity and redundancy only if work is distributable. Shared state, database limits, hot keys, and coordination prevent perfect linear scaling.

## Stateless compute

Stateless app instances can be added/removed freely because durable state is external. The common shape is `LB → stateless app pool → shared data layer`.

## Bottlenecks move

Scaling app CPU may expose the database; adding cache may expose network; sharding may expose cross-shard coordination. Real design is iterative: measure, find bottleneck, change, measure again.

## Autoscaling

Autoscaling signals should represent the constrained resource. CPU can work for CPU-bound services; queue depth or oldest-job age can be better for workers. Blind CPU scaling can miss I/O or downstream bottlenecks.

## Headroom

Operating at 100% utilization leaves no room for bursts, failover, or deployment. Capacity planning includes reserve capacity for abnormal conditions.

## Worked example — a CPU-bound API

A single server reaches 90% CPU. A larger machine may be the simplest first move. As growth continues, run multiple stateless instances behind a balancer. Once the database becomes dominant, further app scaling no longer helps; optimize/index/cache/replicate/partition based on the database workload.

## Interview lens

Describe evolution: “I would start simple; when X becomes the bottleneck, introduce Y.” This is stronger than drawing the final complex system immediately.

## What to remember

Vertical scaling buys simplicity; horizontal scaling buys capacity/redundancy at distributed-systems cost. Scale the bottleneck, not the diagram.

## Review after reading

1. Scale up vs out?
2. Why stateless?
3. Why doesn't adding app servers scale DB writes?
4. What is a good autoscaling metric?
5. Why keep headroom?

## Deeper study notes

### Scalability is not the same as availability

Adding instances can improve both, but they are distinct goals. Ten app servers behind one database can survive one app failure yet still have a single data failure domain. Conversely, a highly available two-node service may not have enough capacity for a 100× traffic increase.

### Amdahl-style thinking

Only the bottlenecked part benefits from scaling. If requests spend 5% of time in app CPU and 95% waiting on one serialized database lock, adding 20 app instances barely helps. Identify the constrained resource before deciding where to add capacity.

### Horizontal scale introduces coordination

Once work spans machines, you face partition ownership, retries, duplicate processing, clocks, distributed locks, consistency, and deployment coordination. That complexity has a cost. Prefer one larger database or one modular service while it comfortably meets requirements.

### Scale by dimension

A service may scale reads and writes differently. Read replicas or caches scale reads; sharding may scale writes/storage; CDN scales byte delivery; worker pools scale background jobs; connection gateways scale long-lived sockets. “Horizontal scaling” is not one mechanism.

### Common mistakes

- Assuming scale-out is always cheaper or simpler than scale-up.
- Scaling a tier that is not the bottleneck.
- Ignoring headroom for failover.
- Autoscaling workers without checking downstream capacity.
- Assuming adding replicas increases single-key write throughput.


## Important interview ideas

> **Important:** Scaling is bottleneck-specific. “Add more servers” is not a complete answer until you identify which resource is saturated and whether the workload can be divided.

### Vertical scaling first can be correct

Interview answers often jump too quickly to distribution. If a database has 200 GB of data and moderate traffic, one well-sized primary plus replicas may be simpler and safer than sharding. Vertical scaling has advantages:

- no cross-shard queries;
- simpler transactions;
- easier operations;
- fewer failure modes.

Use horizontal distribution when one node approaches a real capacity, availability, or geographic limit.

### Scaling dimensions

Different components scale differently:

- app CPU → add stateless app instances;
- read-heavy DB → cache/read replicas;
- write/storage DB → partition/shard;
- media egress → CDN;
- background jobs → worker pool;
- long-lived connections → connection gateways;
- expensive search → partitioned search index.

This is a much stronger answer than “horizontal scaling everywhere.”

### Utilization and queueing

A system near 100% utilization behaves poorly because small bursts create queues. Latency rises sharply as capacity is exhausted. Keep operational headroom for:

- burst traffic;
- one-zone failure;
- rolling deployment;
- retry load;
- unexpected skew.

### Scale and availability are separate

Ten app servers behind one database scale compute and survive app-instance loss, but the database can remain a single point of failure. Conversely, two replicated servers may be highly available but have insufficient capacity. Discuss both dimensions.

## Worked scenario — read-heavy catalog

Start:

```text
App → SQL DB
```

At 10× reads, the DB becomes CPU-bound on repetitive product lookups. First add a distributed cache for hot products. If cache misses/read queries still pressure the primary, add read replicas for stale-tolerant reads. Only when storage/write volume or primary capacity becomes the bottleneck do you consider partitioning.

This is an evolution path, not a prebuilt “internet scale” diagram.

## Interview questions and model answers

### Q1. “When would you scale vertically?”

When the workload still fits comfortably on a larger machine and distribution would add unnecessary complexity. Databases in particular often benefit from vertical scaling before sharding because transactions and queries remain simple.

### Q2. “What prevents linear horizontal scaling?”

Shared bottlenecks and coordination: one database, one lock, one hot key, serialized work, network limits, or cross-node synchronization. If 95% of latency is one serialized DB operation, adding application servers changes little.

### Q3. “What should autoscaling use?”

A signal tied to the constrained resource: CPU for CPU-bound compute, queue depth/age for workers, active connections for gateways, request rate or latency where appropriate. CPU alone can miss I/O-bound saturation.

### Q4. “What does 10× scaling discussion look like?”

Identify the first component that reaches a capacity limit, then evolve only that path. I might say, “At 10×, the product cache is still fine, but write volume exceeds one DB's capacity, so I partition by product ID while keeping the API tier unchanged.”

## Common mistakes to avoid

- “Horizontal scaling is always better.”
- Scaling a non-bottleneck tier.
- Ignoring single points of failure.
- Running with no capacity headroom.
- Assuming stateless compute solves state scaling.
- Adding sharding before indexes/caching/read replicas are evaluated.

## Short revision note

Think **resource → bottleneck → scale mechanism → new bottleneck → trade-off**. Scaling should be evolutionary.

## Topics to revise

- [ ] vertical vs horizontal
- [ ] stateless compute
- [ ] bottleneck identification
- [ ] autoscaling signals
- [ ] capacity headroom
- [ ] read vs write scaling
- [ ] scale vs availability
- [ ] evolution at 10×

## Interview-ready synthesis

### A strong 60–90 second explanation

I scale the constrained resource rather than every box. Vertical scaling is a valid first step when it preserves simplicity. Horizontal scaling is useful when work can be divided and one machine reaches capacity or availability limits. After each change I look for the next bottleneck, because application, database, cache, queue, connection, and media layers scale differently.

### How this topic connects to the wider system

- Performance: adding capacity helps only the saturated resource.
- Reliability: redundancy and capacity are distinct; both must be considered.
- Operations: horizontal scale introduces coordination, routing, and deployment complexity.
- Cost: one larger machine can be cheaper operationally than premature distributed architecture.

### Revision flashcards with answers

**Scale up?**  
Use a larger/faster machine.

**Scale out?**  
Add more machines and distribute work.

**Why doesn't app autoscaling fix DB?**  
The database remains a shared bottleneck.

**Good worker autoscaling metric?**  
Queue age/depth often reflects demand better than CPU for I/O-bound workers.

**Why headroom?**  
To absorb bursts, failover, retries, and deployments without saturation.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- News feed: scale read path with cache/materialization before sharding the source post DB.
- Video: metadata servers scale horizontally, but delivery scale is mainly CDN bandwidth rather than app CPU.
- Logging: worker/consumer scale is limited by downstream storage throughput; adding consumers can simply move the bottleneck.

### When not to overuse this idea

Do not shard or microservice a system that fits comfortably on one reliable database. Distributed scaling pays permanent complexity.

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

**Next:** Lesson 07 — Proxy, Reverse Proxy & API Gateway
