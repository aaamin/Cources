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
