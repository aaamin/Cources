# Session 5 — Load Balancing

## Outcome

You should be able to explain why load balancers exist, distinguish L4 and L7 balancing conceptually, choose common routing strategies, understand health checks, draining, stickiness, regional/global layers, and reason about load-balancer failure.

## Why This Matters

Once a service has multiple replicas, clients need a way to reach healthy instances without knowing every server. Load balancing provides traffic distribution and a stable entry point.

```text
Clients
   ↓
Load Balancer
 ┌─┼─┐
A  B  C
```

It supports horizontal scaling, health-based routing, rolling deployments, and failover.

## L4 vs L7

### Layer 4 conceptually
Routes based on transport-level information such as IP/port/connection.

Strengths:
- efficient;
- works for many TCP/UDP protocols;
- does not need to understand HTTP semantics.

### Layer 7 conceptually
Understands application protocol such as HTTP.

Can route by:
- host;
- path;
- headers;
- cookies;
- methods.

Example:

```text
/api/images/* → Media API pool
/api/orders/* → Order API pool
```

Do not obsess over OSI details in the interview. Explain capability and trade-off.

## Common Routing Algorithms

### Round robin
Send requests across instances in sequence.

Good when instances/requests are similar.

### Least connections
Prefer the instance with fewer active connections.

Useful when requests/connections have variable duration.

### Weighted routing
Send more traffic to larger/healthier pools.

Useful for:
- heterogeneous instances;
- gradual rollout;
- migration.

### Hash-based/sticky routing
Route based on client/session key.

Useful when state/session affinity is unavoidable, but stateless designs are usually easier to scale.

## Health Checks

A load balancer should stop routing to unhealthy instances.

Health checks might be:
- TCP connection;
- HTTP endpoint;
- shallow readiness test.

A health endpoint should not claim healthy when the process cannot serve meaningful traffic.

But avoid making the health check dependent on every optional dependency; otherwise a minor downstream outage can remove all instances.

Different concepts:
- **liveness**: process should be restarted?
- **readiness**: should receive traffic?

## Connection Draining

During deployment or scale-in:

```text
Stop new traffic to instance
        ↓
Allow in-flight requests/connections to finish
        ↓
Terminate instance
```

Without draining, active requests may be cut off.

Long-lived WebSocket connections complicate draining because they may need graceful reconnect.

## Sticky Sessions

A load balancer may keep a user on one server.

Why people use it:
- in-memory session data;
- local caches/state.

Problems:
- uneven load;
- harder failover;
- scaling/rebalancing issues.

Prefer externalizing durable session state when possible.

## Regional Load Balancing

Within a region:

```text
Regional LB
   ↓
Instances across availability zones
```

This spreads traffic and improves resilience to instance/AZ failure.

## Global Traffic Distribution

At global scale:

```text
Users
  ↓
DNS / Anycast / Global traffic manager
  ↓
Region A   Region B   Region C
  ↓          ↓          ↓
Regional LBs
```

Global routing may consider:
- geography;
- latency;
- health;
- capacity;
- residency.

The exact product is less important than the layering concept.

## Load Balancer as a Failure Point

Managed/cloud load balancers are designed for redundancy, but conceptually you should avoid a single custom LB instance:

Bad:

```text
Clients → one nginx box → 100 servers
```

If that box fails, the backend pool is irrelevant.

A production LB tier must itself be highly available.

## Load Balancer Does Not Fix Backend Bottlenecks

Adding ten application servers does not help if every request waits on one overloaded database.

```text
LB → 10 app servers → 1 saturated DB
```

The load balancer solves distribution at one layer, not the whole capacity chain.

## Slow Instances

Round robin assumes roughly equal capacity. If one instance becomes slow but still passes health checks, it may accumulate work.

Possible responses:
- latency-aware/least-connections routing;
- outlier detection;
- application load shedding;
- correct readiness/health signals.

## Worked Example — API Scaling

Initial:
```text
Client → App
```

Need rolling deployments and 3× traffic:

```text
Client
  ↓
L7 Load Balancer
  ↓
App A  App B  App C
  ↓
Shared DB/cache
```

Apps are stateless.

During deploy:
1. mark App A draining;
2. stop new requests to A;
3. finish in-flight traffic;
4. replace;
5. verify health;
6. repeat.

If DB becomes saturated, scaling app instances further may worsen DB pressure. Find the actual bottleneck.

## Small Design Drills

1. Why is least-connections useful for long-lived connections?
2. What can path-based L7 routing do that a simple L4 balancer cannot?
3. Why are sticky sessions usually undesirable?
4. An app instance returns 200 from `/health` but all DB calls time out. Is the health check sufficient?
5. Why doesn't adding app servers always increase system throughput?
6. What is connection draining for?

<details>
<summary>Answer key</summary>

1. It avoids repeatedly sending new connections to instances already holding many active connections.
2. Inspect HTTP path/host/header and route to different pools.
3. They couple users to instances, create imbalance, and complicate failover/rebalancing.
4. Probably not; readiness should represent ability to serve the critical path, but should also avoid depending on every optional dependency.
5. A downstream dependency such as DB/cache may be the bottleneck.
6. Gracefully remove an instance from traffic while allowing existing work to complete.

</details>

## Common Mistakes

- Saying load balancing alone makes a system highly available.
- Ignoring LB redundancy.
- Treating sticky sessions as the normal design.
- Using round robin without considering highly uneven request duration.
- Scaling apps while DB is the bottleneck.
- Confusing global DNS/traffic steering with a regional request LB.
- Forgetting connection draining.

## Must Remember

- **Load balancing enables traffic distribution across replicas.**
- **L4 routes at transport level; L7 understands application protocol.**
- **Health checks decide who should receive traffic.**
- **Drain before terminating active instances.**
- **Stateless services minimize the need for stickiness.**
- **Global and regional load balancing are different layers.**
- **The LB cannot remove a downstream bottleneck.**
- **The load-balancing tier itself must be highly available.**

## Interview Revision Summary

Use:

```text
Global routing
     ↓
Regional LB
     ↓
Stateless service replicas
```

Discuss:
- routing strategy;
- health;
- draining;
- stickiness only if needed;
- failure of instances/AZ;
- actual downstream bottleneck.

## Explain Without Notes

Explain how you would evolve a single API server into a horizontally scaled regional deployment and perform zero/minimal-downtime rolling replacements.

## Completion Checklist

- [ ] I understand L4 vs L7 conceptually.
- [ ] I can explain routing algorithms.
- [ ] I understand health/readiness and draining.
- [ ] I can discuss sticky-session trade-offs.
- [ ] I distinguish regional and global balancing.
- [ ] I remember downstream bottlenecks.
