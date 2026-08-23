# Lesson 07 — Proxy, Reverse Proxy & API Gateway

**Phase:** Fundamentals  
**Session:** 7/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn forward proxies, reverse proxies, API gateways, their overlap with load balancers, and which responsibilities belong at the edge.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Forward proxy

A forward proxy acts on behalf of clients: `Client → Proxy → Internet`. It can enforce outbound policy, hide client addresses, cache, or filter traffic.

## Reverse proxy

A reverse proxy acts on behalf of servers: `Internet → Reverse Proxy → Backends`. It can hide topology, terminate TLS, route, compress, cache, and enforce request limits.

## API gateway

An API gateway is a policy-rich edge for application APIs: routing, auth, quotas, transformation, observability, aggregation. It should normally handle cross-cutting concerns, not become the central owner of business logic.

## Overlap with load balancing

Products often combine balancing, proxying, and gateway features. Conceptually, a load balancer emphasizes distribution, a reverse proxy server-side mediation, and a gateway API policy. Explain capabilities rather than fighting over labels.

## Edge responsibilities

Useful edge functions include TLS termination, authentication, rate limiting, routing, request-size limits, and trace IDs. Authorization tied to business data often still belongs in the service that owns that data.

## Service discovery — an important companion concept

Once a system has many service instances, callers need a way to find healthy destinations. Hard-coding IP addresses does not work because instances are created, removed, rescheduled, or replaced.

At interview depth, understand two broad models:

```text
Client-side discovery
caller → registry/DNS → choose instance → service

Server-side discovery
caller → LB/proxy → registry/backend pool → service
```

A service registry or DNS-based mechanism tracks discoverable endpoints. Health checks remove dead instances. In many managed environments this is hidden behind a load balancer, service name, or orchestration platform.

> **Important:** Service discovery answers **where is the service?** Load balancing answers **which healthy instance should receive this request?** The capabilities often work together.

Do not make service discovery a separate box unless the prompt needs internal dynamic routing. For a simple monolith or a handful of fixed services, it may be invisible infrastructure detail.

## Worked example — public APIs for several services

An API gateway accepts `/users`, `/orders`, and `/payments`, authenticates once, and routes to the owning service. Internal services can still use their own load balancing. The gateway centralizes external policy without becoming the system of record.

## Interview lens

If you draw gateway + proxy + load balancer as three boxes, be able to justify why they are distinct. Otherwise use one conceptual edge layer.

## What to remember

Think in responsibilities: client proxying, server proxying, traffic distribution, and API policy are related but different.

## Review after reading

1. Who does a forward proxy represent?
2. Who does a reverse proxy represent?
3. Common gateway duties?
4. Why do products overlap?
5. What business logic should stay out of the gateway?

## Deeper study notes

### Place policy at the right boundary

Cross-cutting policy belongs near the edge when it can be applied uniformly: TLS termination, request-size limits, coarse authentication, quotas, routing, trace IDs. Business authorization often needs resource state and belongs in the owning service. For example, the gateway can verify a user's token, but the File Service decides whether that user may read file `f123`.

### Avoid the “god gateway”

A gateway that contains pricing rules, order orchestration, user preferences, and data joins becomes a bottleneck and coupling point. The edge should simplify clients and centralize infrastructure policy, not absorb all business behavior.

### Reverse proxy and caching

Reverse proxies can cache public responses, compress content, terminate connections, and shield origins. This makes them useful even in monolith architectures; gateways are not only for microservices.

### BFF concept

A Backend-for-Frontend tailors API aggregation to one client type, such as mobile vs web. It can reduce round trips and client complexity, but it introduces another service and should not become the canonical source of business data.

### Common mistakes

- Duplicating authorization logic inconsistently between gateway and service.
- Assuming a gateway eliminates the need for service-level load balancing.
- Putting every business workflow in the gateway.
- Confusing outbound forward proxy with inbound reverse proxy.


## Important interview ideas

> **Important:** These terms overlap in real products. In interviews, focus on responsibilities, not vendor labels.

### Responsibility map

```text
Forward proxy  → represents clients to the outside world
Reverse proxy  → represents backend servers to clients
Load balancer  → distributes traffic among backends
API gateway    → enforces/API-level policy and routing
```

One product can implement several of these at once. The conceptual distinction helps you explain **why the component is present**.

### Good gateway responsibilities

Cross-cutting concerns are natural at the edge:

- TLS termination;
- authentication token verification;
- coarse authorization such as API scopes;
- rate limits and quotas;
- request-size limits;
- routing/version selection;
- correlation IDs and request logs.

Resource-specific business authorization usually belongs in the owning service, because only that service knows the object relationship. A gateway can verify “this is Alice,” while Drive must decide “Alice may read file 123.”

### Avoid the “smart gateway” trap

If every workflow and business rule moves into the gateway, it becomes a centralized monolith that couples all services. Keep edge policy general and keep domain logic near domain state.

### Reverse proxy and caching

A reverse proxy may cache responses, compress content, terminate TLS, or serve static files. This can reduce app work. But personalized responses require careful cache keys to avoid leaking one user's data to another.

## Worked scenario — API edge

```text
Mobile/Web
    ↓
API Gateway
  - TLS
  - JWT validation
  - rate limit
  - route
    ├→ User Service
    ├→ Order Service
    └→ Payment Service
```

The gateway does not directly modify order tables. It routes authenticated requests. Order Service performs object-level authorization and business validation.

## Interview questions and model answers

### Q1. “API gateway vs load balancer?”

A load balancer's core job is distributing traffic. An API gateway operates at the API layer and may authenticate, rate-limit, transform, and route by endpoint. In practice one managed edge can do both. I would draw separate boxes only if their responsibilities or scaling/failure domains matter.

### Q2. “Should authentication happen only at the gateway?”

The gateway can verify tokens centrally, but sensitive services should still trust identity through a secure internal mechanism and enforce resource authorization themselves. Defense in depth is useful; the gateway should not be the only barrier against internal misrouting.

### Q3. “What is a BFF?”

A Backend-for-Frontend is an API layer tailored to one client type, such as mobile, that aggregates calls or shapes responses. It can reduce chatty client behavior, but adds another maintained service and should not own core domain truth.

### Q4. “When do I not need an API gateway?”

A small monolith or one-service application can expose HTTPS directly through a reverse proxy/load balancer. Do not add a gateway merely because the diagram looks more “distributed.”

## Common mistakes to avoid

- Treating gateway, proxy, and LB as mutually exclusive product categories.
- Moving business logic into the gateway.
- Assuming token validation equals object authorization.
- Caching personalized responses with unsafe keys.
- Adding several edge layers without a reason.

## Short revision note

Remember the **responsibility**, not the box name. Edge layers handle routing and cross-cutting policy; domain services own business rules and data.

## Topics to revise

- [ ] forward proxy
- [ ] reverse proxy
- [ ] load balancer
- [ ] API gateway
- [ ] TLS termination
- [ ] authn vs authz at edge
- [ ] rate limiting
- [ ] BFF concept
- [ ] service discovery
- [ ] client-side vs server-side discovery

## Interview-ready synthesis

### A strong 60–90 second explanation

I distinguish edge components by responsibility. A reverse proxy represents servers, a load balancer distributes requests, and an API gateway applies API-level policy such as authentication, quotas, version routing, and observability. One product may implement several roles, so I avoid drawing duplicate layers unless their responsibilities truly differ.

### How this topic connects to the wider system

- Security: gateway can validate identity, but resource authorization belongs near domain state.
- Reliability: central edge policy must itself be highly available and have safe failure behavior.
- Performance: TLS termination, compression, caching, and aggregation can reduce backend work.
- Maintainability: keeping business logic out of the gateway avoids centralized coupling.

### Revision flashcards with answers

**Forward proxy?**  
An intermediary acting on behalf of clients toward external services.

**Reverse proxy?**  
An intermediary clients contact that forwards to backend servers.

**Gateway vs LB?**  
Gateway emphasizes API policy/routing; LB emphasizes traffic distribution, though products overlap.

**Where authorize file access?**  
The file-owning service, because it knows permissions; gateway may only verify identity/token.

**Need gateway for monolith?**  
Not necessarily; a reverse proxy/LB may be sufficient.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Public API: gateway validates tokens/rate limits, then routes to domain services that own authorization/business logic.
- Mobile BFF: aggregates several backend calls into one client-oriented response when mobile round trips are costly.
- Internal service mesh: may provide proxying/mTLS/observability, but is infrastructure detail unless the interview asks for it.

### When not to overuse this idea

Do not draw gateway + reverse proxy + LB + service mesh as separate boxes unless each has a distinct responsibility relevant to the prompt.

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

**Next:** Lesson 08 — CDN & Edge Caching
