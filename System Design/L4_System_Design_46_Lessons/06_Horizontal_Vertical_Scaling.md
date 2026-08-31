# Session 6 — Horizontal & Vertical Scaling

## Outcome

You should be able to distinguish scale-up from scale-out, explain why stateless services scale more easily, identify CPU/memory/network/downstream bottlenecks, reason about autoscaling and headroom, and describe how a system evolves rather than adding components blindly.

## Vertical Scaling — Scale Up

Increase capacity of one machine:

```text
2 CPU / 4 GB
     ↓
8 CPU / 32 GB
```

Advantages:
- simple;
- no distributed coordination introduced;
- useful for databases or workloads that benefit from larger memory/CPU.

Limits:
- hardware ceiling;
- larger failure blast radius;
- often nonlinear cost;
- resizing may require disruption depending on platform.

Vertical scaling is not “bad.” It is often the simplest first step.

## Horizontal Scaling — Scale Out

Add more instances:

```text
1 server
   ↓
LB
├─ Server A
├─ Server B
└─ Server C
```

Advantages:
- more aggregate capacity;
- redundancy;
- independent replacement;
- elastic scaling.

Costs:
- need load distribution;
- shared/durable state must be handled;
- downstream systems may become bottlenecks;
- coordination may appear.

## Stateless Application Services

A stateless service does not require a specific replica to remember durable per-user state.

State can live in:
- database;
- distributed cache;
- object store;
- token/session store.

Then any request can go to any replica.

This makes:
- autoscaling;
- failover;
- rolling deploys;
- load balancing

much easier.

## Stateful Components

Some systems inherently hold state:
- DB shards;
- WebSocket connection gateways;
- stream partitions;
- cache nodes.

Stateful scaling requires:
- ownership;
- replication;
- rebalancing;
- recovery.

Do not force everything to be stateless; understand the cost when it isn't.

## Bottleneck Thinking

System throughput is constrained by the weakest required stage.

```text
LB → App → Cache → DB → External API
```

Possible bottlenecks:
- CPU;
- memory;
- garbage collection;
- network bandwidth;
- connection pools;
- disk IOPS;
- DB locks;
- database CPU;
- external API rate limit;
- queue consumer throughput.

Scaling the wrong tier can make things worse.

## CPU Bottleneck

Symptoms:
- sustained high CPU;
- increasing request latency;
- run queue;
- throughput stops increasing.

Response:
- optimize expensive work;
- cache/precompute;
- scale up/out;
- move background CPU work off request path.

## Memory Bottleneck

Symptoms:
- high GC/heap pressure;
- swapping/OOM;
- cache eviction;
- process restarts.

Response depends on cause:
- memory leak;
- oversized cache;
- high per-connection state;
- workload needs more memory.

## Network Bottleneck

Large media or high response volume can saturate network long before CPU.

Response:
- CDN;
- direct object-storage transfer;
- compression;
- regional placement;
- scale network endpoints.

## Downstream Bottleneck

If apps scale from 5 to 50 instances, DB connection attempts may increase 10×.

This can overload the database.

Horizontal scaling requires coordinated capacity limits:
- connection pools;
- admission control;
- cache;
- queue;
- DB scaling.

## Autoscaling

Autoscaling changes instance count based on signals.

Possible signals:
- CPU;
- request rate;
- queue depth;
- latency;
- custom saturation metric.

For background workers, **queue depth / age** is often more meaningful than CPU.

### Problem with reactive autoscaling

If instances take minutes to start, a sudden burst may overload the system before scaling finishes.

Solutions:
- headroom;
- scheduled scaling for predictable peaks;
- queues to absorb work;
- fast startup;
- conservative thresholds.

## Capacity Headroom

Do not run continuously at 100% theoretical capacity.

Reasons:
- traffic forecast error;
- instance failure;
- deploys;
- burst;
- noisy workload;
- downstream slowdown.

If one instance handles 1,000 RPS under test, planning to run it at exactly 1,000 RPS in production is risky.

## Failure Capacity

Suppose you need 3 instances at normal peak.

If one fails:
- remaining 2 must survive until replacement.

N+1 or percentage headroom may matter depending on availability target.

## Scale by Bottleneck, Not User Count

“1 million users” alone says little.

1M users who log in once/month can be easier than 10k users streaming high-rate telemetry.

Estimate actions, concurrency, payload, and skew.

## Evolution Example

Stage 1:
```text
App + DB
```

Stage 2:
```text
LB → App replicas → DB
```

Stage 3, read-heavy:
```text
LB → App replicas → Cache → DB
```

Stage 4, background work:
```text
App → Queue → Workers
```

Stage 5, DB limits:
- better indexes/queries;
- read replicas;
- partitioning if truly necessary.

Do not jump straight to Stage 5.

## Small Design Drills

1. Why can a stateless service scale horizontally more easily?
2. App CPU is 30%, DB CPU is 95%. Should you add app instances?
3. A worker backlog grows while CPU is 20%. What autoscaling metric might be better?
4. Why is 100% target utilization dangerous?
5. A media API has 2% CPU but network is saturated. What kind of scaling/architecture change helps?
6. Is vertical scaling ever reasonable?

<details>
<summary>Answer key</summary>

1. Any replica can serve any request; no user/session ownership must be migrated.
2. Not as the first response; DB is the bottleneck and more apps may add pressure.
3. Queue depth and oldest-message age.
4. No room for failures, bursts, deploys, or forecast errors.
5. CDN/object-storage direct delivery, more network capacity, separation of media serving.
6. Yes. It is simple and often useful until limits/cost/availability justify scale-out.

</details>

## Common Mistakes

- Scaling by user count alone.
- Assuming scale-out removes all bottlenecks.
- Ignoring connection pool growth.
- Targeting 100% utilization.
- Autoscaling only on CPU.
- Ignoring startup time.
- Treating vertical scaling as inherently wrong.
- Making stateful services “stateless” by hiding state problems.

## Must Remember

- **Scale up = bigger node; scale out = more nodes.**
- **Stateless request handlers are easier to scale and replace.**
- **The real bottleneck determines throughput.**
- **More app servers can overload the database.**
- **Choose autoscaling metrics that reflect workload.**
- **Keep headroom for burst/failure/deploys.**
- **Queues can absorb bursts but do not create infinite capacity.**
- **Evolve architecture only when a limit justifies it.**

## Interview Revision Summary

Ask:

```text
What is saturated?
CPU?
Memory?
Network?
Pool?
DB?
External dependency?
Queue consumer?
```

Then choose the smallest justified scaling move.

## Explain Without Notes

You have an API service capable of 2k RPS per instance, a DB capable of 5k QPS, and each request makes two DB queries. Explain why adding ten app servers does not make the whole system capable of 20k RPS.

## Completion Checklist

- [ ] I distinguish scale-up and scale-out.
- [ ] I understand stateless vs stateful scaling.
- [ ] I can identify likely bottleneck classes.
- [ ] I understand autoscaling signals and delay.
- [ ] I consider headroom and failure capacity.
