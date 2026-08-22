# Session 07 — Proxy, Reverse Proxy & API Gateway

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Distinguish the common traffic-management components and know what responsibilities belong at the edge.

## What You Need to Read / Learn

- Forward proxy: acts on behalf of clients.
- Reverse proxy: acts in front of servers.
- API gateway: application-facing entry point that may provide routing, auth, quotas, transformations, and aggregation.
- TLS termination, request routing, header manipulation, and compression.
- Authentication versus authorization at gateway/service boundaries.
- Rate limiting and quotas at the edge.
- Request aggregation/BFF patterns and the risk of creating a giant gateway.
- Service discovery conceptually for routing to dynamic backend instances.

## What You Need to Do

- [ ] Draw a request path that includes CDN, load balancer/reverse proxy, API gateway, and backend service. Explain why each exists.
- [ ] Decide where to enforce authentication, rate limiting, and business authorization.
- [ ] List two responsibilities that should generally stay out of the gateway.

## **Must Remember for the Interview**

- **A reverse proxy represents servers; a forward proxy represents clients.**
- **An API gateway is a logical application entry point, not automatically a load balancer replacement.**
- **Edge layers should handle cross-cutting concerns, not absorb all business logic.**
- **Authentication can be centralized; resource-level authorization often still belongs near the owning service.**
- **Every extra hop adds latency and operational dependency.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Forward proxy → client-side intermediary. Reverse proxy → server-side intermediary.**
- **API gateway → routing + cross-cutting API concerns.**
- **Keep business invariants in the service/data owner, not only at the edge.**
- **Use gateways intentionally; avoid a distributed monolith hidden behind one gateway.**
- **Explain responsibilities, not just component names.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain forward proxy vs reverse proxy?
- [ ] Can I explain API gateway vs load balancer?
- [ ] Can I place authN and authZ responsibly?
- [ ] Can I explain why gateway business logic can be dangerous?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 7/46  
**Next:** Session 8
