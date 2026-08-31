# Session 4 — Application Architecture & Service Boundaries

## 1. Must Learn

### Monolith, modular monolith, microservices
- **Understand:** Understand what each architecture style means and why a modular monolith is often a strong starting point.
- **Decision/trade-off:** Simplicity/local transactions vs independent deployment/scaling/ownership.

### Service boundaries
- **Understand:** Define boundaries around cohesive domains with clear ownership; understand coupling and database ownership.
- **Decision/trade-off:** Independent evolution vs distributed coordination and duplicated data.

### Stateless vs stateful services
- **Understand:** Know why stateless app servers are easier to scale horizontally and where state can live instead.
- **Decision/trade-off:** Scalability/failover simplicity vs locality/session affinity.

### Synchronous vs asynchronous communication
- **Understand:** Understand request/response service calls vs queues/events.
- **Decision/trade-off:** Immediate result/simple flow vs decoupling, buffering, eventual completion.

### Distributed-service costs
- **Understand:** Know microservices add network failure, deployment, observability, and consistency complexity.
- **Decision/trade-off:** Independent ownership/scaling vs operational complexity.

### Request lifecycle & dependency chains
- **Understand:** Trace a request through app servers, connection pools, dependencies, timeouts, and failure propagation.
- **Decision/trade-off:** Convenient synchronous composition vs latency/cascading-failure risk.

## 2. Should Know

- Shared database vs database-per-service at design level.
- Independent deployment and independent scaling as reasons—not goals by themselves—for splitting services.
- Connection pools and dependency latency as common bottlenecks.

## 3. Recognition Only

- SOA terminology
- Service discovery internals
- Service mesh internals

## 4. Important Comparisons

- Monolith vs modular monolith vs microservices.
- Stateful vs stateless application tier.
- Synchronous vs asynchronous communication.
- Shared database vs database ownership per service.
- High cohesion vs tightly coupled cross-service boundaries.

## 5. Important Interview Questions

1. When should I split a service instead of keeping a modular monolith?
2. What problem do microservices solve, and what complexity do they add?
3. Why do stateless services scale more easily?
4. When should an operation move off the synchronous request path?
5. What happens when one dependency in a synchronous chain becomes slow?
6. Who owns each piece of data?

## 6. Common Interview Mistakes

- **Microservices by default** → Start simple; split only for clear ownership, scaling, deployment, or isolation needs.
- **Service per noun/table** → Choose cohesive domain boundaries, not arbitrary tiny services.
- **Shared state in app memory** → Use external/shared state if requests can land on any server.
- **Long synchronous chains** → Recognize latency and cascading-failure risk.
- **Ignoring data ownership** → Make source-of-truth ownership explicit.

## 7. Communication

### Important Vocabulary

monolith, modular monolith, microservice, service boundary, cohesion, coupling, stateless, stateful, synchronous, asynchronous, data ownership, dependency

### Useful Interview Phrases

- “I’d start with a modular monolith unless independent scaling or ownership justifies a split.”
- “This boundary keeps related logic and data ownership together.”
- “Moving this work asynchronous reduces request-path coupling, but completion becomes eventual.”

### Important Questions to Ask the Interviewer

- **Question:** “Do these domains need independent teams/deployment?”  
  **Why it matters:** Helps justify service separation.
- **Question:** “Do any components scale very differently?”  
  **Why it matters:** May justify independent service scaling.
- **Question:** “Must this operation complete before responding to the user?”  
  **Why it matters:** Determines sync vs async.

## 8. ⭐ Must Remember

1. Microservices are a trade-off, not a maturity level.
2. Prefer cohesive boundaries and explicit data ownership.
3. Stateless app tiers scale more easily.
4. Every synchronous dependency adds latency/failure risk.
5. Use async work when immediate completion is unnecessary.
6. Start simple; split when requirements justify it.

## 9. Study Priority

1. Study first: architecture styles and service-boundary reasoning.
2. Study next: stateless services, data ownership, sync vs async.
3. Finish with: request lifecycle, dependency latency, and failure propagation.

## 10. Revision Checklist

- [ ] Choose monolith/modular monolith/microservices for a scenario and defend it.
- [ ] Draw clear service/data ownership boundaries.
- [ ] Explain stateless scaling.
- [ ] Choose sync vs async communication.
- [ ] Trace failure propagation through a request chain.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
