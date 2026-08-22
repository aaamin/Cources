# Session 05 — Load Balancing

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn how traffic is distributed across healthy compute and how load balancers participate in availability.

## What You Need to Read / Learn

- L4 versus L7 load balancing conceptually: transport-level versus application-aware routing.
- Algorithms: round robin, weighted round robin, least connections, hashing/sticky routing.
- Health checks and removing unhealthy targets.
- Connection draining during deployment or instance removal.
- TLS termination at the load balancer/reverse proxy.
- Sticky sessions and their operational cost.
- Regional load balancing versus global traffic management.
- Load balancer redundancy: avoid treating one logical LB as one physical single point of failure.

## What You Need to Do

- [ ] Choose an algorithm for short HTTP requests, long WebSocket connections, and heterogeneous servers.
- [ ] Explain what happens when one application instance fails mid-request.
- [ ] Explain how to deploy a new version without abruptly dropping all connections.

## **Must Remember for the Interview**

- **A load balancer distributes traffic; it does not automatically remove downstream bottlenecks.**
- **Health checks must reflect whether an instance can actually serve traffic.**
- **Least-connections can fit long-lived connections better than naive round robin.**
- **Sticky sessions reduce flexibility and should have a clear reason.**
- **Connection draining helps safe deployment and instance retirement.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **L4 routes using connection/network information; L7 can route using HTTP/application information.**
- **Load balancers enable horizontal compute scaling and failover.**
- **LB choice depends on request duration, server capacity, and session behavior.**
- **Never forget health checks and graceful removal.**
- **The database/cache can still be the bottleneck after adding app servers.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain L4 vs L7?
- [ ] Can I choose a load-balancing algorithm for WebSockets?
- [ ] Can I explain sticky sessions and their downside?
- [ ] Can I explain connection draining?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 5/46  
**Next:** Session 6
