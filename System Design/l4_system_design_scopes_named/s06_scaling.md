# Session 6 — Horizontal & Vertical Scaling

## 1. Must Learn

### Scale up vs scale out
- **Understand:** Understand adding resources to one machine vs adding machines.
- **Decision/trade-off:** Simplicity vs horizontal elasticity/fault tolerance/coordination.

### Stateless horizontal scaling
- **Understand:** Know why stateless app servers can be replicated behind a load balancer.
- **Decision/trade-off:** Easy distribution/failover vs externalizing shared state.

### Bottleneck identification
- **Understand:** Know scaling one tier only moves the bottleneck to databases, caches, queues, network, or external dependencies.
- **Decision/trade-off:** Targeted scaling vs wasteful blanket scaling.

### Autoscaling & triggers
- **Understand:** Understand scaling based on CPU, memory, request rate, queue depth, or other workload signals.
- **Decision/trade-off:** Responsiveness/headroom vs oscillation, delay, and cost.

### Session/shared-state problems
- **Understand:** Understand why server-local session state complicates scale out.
- **Decision/trade-off:** Local simplicity vs affinity/failover constraints.

### Single points of failure & capacity headroom
- **Understand:** Identify non-redundant critical components and reserve capacity for bursts/failures.
- **Decision/trade-off:** Cost efficiency vs resilience.

## 2. Should Know

- Scaling limits of dependencies and connection pools.
- Cold-start/warm-up effects when autoscaling.
- Vertical scaling can be a practical first step before distributed complexity.

## 3. Recognition Only

- Scale-to-zero
- Predictive autoscaling
- NUMA/hardware internals

## 4. Important Comparisons

- Vertical vs horizontal scaling.
- Stateful vs stateless scaling.
- Reactive autoscaling vs fixed headroom.
- Scaling app tier vs scaling the actual bottleneck.

## 5. Important Interview Questions

1. When should I scale up before scaling out?
2. What prevents this tier from scaling horizontally?
3. Where does session state live?
4. What breaks after I add more application servers?
5. Which metric should trigger scaling?
6. What happens during a sudden spike before new capacity is ready?

## 6. Common Interview Mistakes

- **“Just add servers”** → Identify whether the bottleneck is actually in that tier.
- **Keeping critical session state locally** → Externalize or intentionally use affinity with known trade-offs.
- **No headroom** → Plan for spikes, deployments, and instance failure.
- **CPU-only autoscaling** → Use a metric aligned with the actual workload.
- **Ignoring dependency capacity** → More app servers can overload databases or downstream services.

## 7. Communication

### Important Vocabulary

scale up, scale out, stateless, autoscaling, bottleneck, saturation, headroom, single point of failure, shared state, capacity

### Useful Interview Phrases

- “I’d scale the stateless tier horizontally behind a load balancer.”
- “Before adding capacity, I’d verify which resource is actually saturated.”
- “Autoscaling helps sustained growth, but I still need headroom for sudden bursts.”

### Important Questions to Ask the Interviewer

- **Question:** “Is the workload bursty or predictable?”  
  **Why it matters:** Changes autoscaling/headroom strategy.
- **Question:** “Does the service keep local session state?”  
  **Why it matters:** Determines horizontal-scaling complexity.
- **Question:** “Which resource hits saturation first?”  
  **Why it matters:** Identifies the real bottleneck.

## 8. ⭐ Must Remember

1. Scale up is simpler; scale out offers more elasticity and fault tolerance.
2. Stateless services are easiest to scale horizontally.
3. Scaling one tier shifts bottlenecks elsewhere.
4. Autoscaling is not instantaneous.
5. Keep headroom for bursts and failures.
6. Shared state and dependencies often become the next scaling limit.

## 9. Study Priority

1. Study first: vertical vs horizontal and stateless scaling.
2. Study next: bottlenecks, shared state, dependency limits.
3. Finish with: autoscaling triggers, headroom, and failure capacity.

## 10. Revision Checklist

- [ ] Evolve one server into a horizontally scaled app tier.
- [ ] Explain what state must move out of process.
- [ ] Identify the next bottleneck.
- [ ] Choose a useful scaling trigger.
- [ ] Discuss headroom and failure capacity.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
