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
