# Session 06 — Horizontal & Vertical Scaling

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand when to add resources to a machine, when to add machines, and where scaling stops helping.

## What You Need to Read / Learn

- Vertical scaling: larger CPU, RAM, disk, or network on one machine.
- Horizontal scaling: add instances and distribute load.
- Stateless services as a prerequisite for easy horizontal compute scaling.
- Autoscaling signals: CPU, request rate, queue depth, latency, custom business metrics.
- Shared state and why it can become the next bottleneck.
- Amdahl-like intuition: scaling one tier does not help if another tier dominates.
- Single points of failure and failure domains.
- Capacity headroom for bursts and failover.

## What You Need to Do

- [ ] Evolve a single application+database server into a horizontally scalable application tier.
- [ ] Identify what breaks if you autoscale workers only on CPU but the queue is growing.
- [ ] For a read-heavy service, list the likely bottleneck after app servers are scaled out.

## **Must Remember for the Interview**

- **Horizontal scaling is not free: coordination, data partitioning, and operational complexity increase.**
- **Stateless compute scales more cleanly than stateful compute.**
- **Autoscaling should follow the resource or backlog that predicts user pain.**
- **Scale the bottleneck, not the component that is easiest to duplicate.**
- **Keep headroom for failure and bursts.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Scale up for simplicity until limits/cost justify scaling out.**
- **Scale out compute with load balancing; state usually needs a separate strategy.**
- **Watch CPU, memory, network, connection pools, database capacity, and queues.**
- **Autoscaling is reactive unless pre-scaling is used for predictable events.**
- **A distributed architecture buys capacity/availability at the cost of complexity.**

## Self-Test Before Marking This Session Complete

- [ ] Can I compare vertical and horizontal scaling?
- [ ] Can I explain why statelessness helps?
- [ ] Can I choose a useful autoscaling metric?
- [ ] Can I identify the next bottleneck after scaling app servers?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 6/46  
**Next:** Session 7
