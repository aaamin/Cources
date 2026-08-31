# Session 5 — Load Balancing

## 1. Must Learn

### Why load balancing exists
- **Understand:** Understand distributing traffic across multiple healthy instances for scale and availability.
- **Decision/trade-off:** Distribution simplicity vs routing intelligence.

### L4 vs L7 balancing
- **Understand:** Know L4 routes using transport-level information; L7 can route using HTTP/application information.
- **Decision/trade-off:** Lower-layer simplicity/performance vs richer routing/control.

### Routing strategies
- **Understand:** Understand round robin, least connections, and weighted routing conceptually.
- **Decision/trade-off:** Even/simple distribution vs awareness of connection count or heterogeneous capacity.

### Health checks & failover
- **Understand:** Know unhealthy instances should stop receiving new traffic and recovery needs cautious re-entry.
- **Decision/trade-off:** Fast removal vs false positives/flapping.

### Connection draining
- **Understand:** Understand graceful removal so in-flight work can finish.
- **Decision/trade-off:** Fast deploy/failover vs dropped in-flight requests.

### Sticky sessions
- **Understand:** Understand affinity and why it can complicate scaling/failover.
- **Decision/trade-off:** Session locality vs uneven load and reduced flexibility.

### Regional/global balancing
- **Understand:** Know regional LB vs global traffic routing at a conceptual level.
- **Decision/trade-off:** Low latency/locality vs failover and routing complexity.

## 2. Should Know

- Cross-zone/multi-AZ traffic and resilience implications.
- Load balancer redundancy / avoiding it as a single point of failure.
- Connection-oriented workloads may require routing based on connections, not just requests.

## 3. Recognition Only

- Consistent-hash load balancing
- Anycast at recognition level

## 4. Important Comparisons

- L4 vs L7 load balancing.
- Round robin vs least connections vs weighted routing.
- Sticky vs non-sticky sessions.
- Regional vs global load balancing.

## 5. Important Interview Questions

1. Why does horizontal scaling usually require load balancing?
2. How should unhealthy instances be removed from rotation?
3. When is sticky session behavior justified?
4. What changes for long-lived connections?
5. What happens if an entire zone or region fails?

## 6. Common Interview Mistakes

- **Adding many app servers without traffic distribution** → Explain how clients reach healthy instances.
- **Treating health checks as perfect** → Consider transient failure/flapping and graceful recovery.
- **Sticky sessions by default** → Prefer stateless services unless affinity is required.
- **Ignoring LB failure** → Use a highly available managed/redundant balancing layer.
- **Using one routing algorithm universally** → Match strategy to workload.

## 7. Communication

### Important Vocabulary

L4, L7, round robin, least connections, weighted routing, health check, failover, connection draining, sticky session, regional load balancing, global load balancing

### Useful Interview Phrases

- “The load balancer lets me scale the stateless tier horizontally.”
- “I’d remove unhealthy instances from new traffic and drain existing connections.”
- “Sticky sessions solve affinity but make load distribution and failover less flexible.”

### Important Questions to Ask the Interviewer

- **Question:** “Are connections short-lived or persistent?”  
  **Why it matters:** Changes routing and draining concerns.
- **Question:** “Are instances homogeneous?”  
  **Why it matters:** Affects round-robin vs weighted strategies.
- **Question:** “Do we need cross-region failover?”  
  **Why it matters:** Determines whether global routing is needed.

## 8. ⭐ Must Remember

1. Load balancing enables horizontal traffic distribution.
2. L4 and L7 operate at different layers and offer different routing intelligence.
3. Health checks prevent sending new work to unhealthy instances.
4. Drain before terminating when in-flight work matters.
5. Sticky sessions are a trade-off, not the default.
6. Global and regional balancing solve different scopes.

## 9. Study Priority

1. Study first: purpose, L4/L7, core routing algorithms.
2. Study next: health checks, failover, draining.
3. Finish with: sticky sessions and regional/global balancing.

## 10. Revision Checklist

- [ ] Explain why an app-server pool needs a load balancer.
- [ ] Compare L4 and L7.
- [ ] Choose a routing strategy.
- [ ] Explain health checks/draining.
- [ ] Discuss sticky-session and regional-failover trade-offs.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
