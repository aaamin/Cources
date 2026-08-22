# Session 04 — Client–Server Architecture & Request Lifecycle

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand the basic backend request path and where state, latency, and failures appear.

## What You Need to Read / Learn

- Client/server model and request/response lifecycle.
- Application servers: request parsing, authentication, business logic, storage calls, and response creation.
- Stateful versus stateless services and why stateless compute is easier to scale horizontally.
- Connection pools to databases and downstream services.
- Synchronous service-to-service calls and latency accumulation.
- Session state: cookies/tokens versus server-local session memory.
- Timeouts and cancellation propagation conceptually.
- Failure points: client disconnect, server crash, dependency timeout, database overload.

## What You Need to Do

- [ ] Draw a login request lifecycle including auth, application server, cache/database, and response.
- [ ] Redesign a stateful app server so any healthy instance can serve the next request.
- [ ] Identify three places where a request can be duplicated after a timeout.

## **Must Remember for the Interview**

- **Stateless application servers are easier to load-balance and replace.**
- **Remote dependencies turn one request into a distributed workflow with multiple failure points.**
- **Server-local state creates affinity and recovery problems unless deliberately managed.**
- **Connection pools are finite resources and can become bottlenecks.**
- **A timeout tells you the caller stopped waiting; it does not prove the server did not finish the work.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Keep durable/shared state outside replaceable compute when possible.**
- **Synchronous dependency chains increase latency and failure coupling.**
- **Session affinity is a trade-off, not a default.**
- **After a timeout, the operation may be in an unknown state.**
- **Design request paths with bounded dependencies.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain stateless vs stateful services?
- [ ] Can I identify the source of truth in a simple client-server system?
- [ ] Can I explain why a timeout does not mean failure?
- [ ] Can I name common finite resources on an app server?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 4/46  
**Next:** Session 5
