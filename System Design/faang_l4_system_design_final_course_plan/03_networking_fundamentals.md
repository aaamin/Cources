# Session 03 — Networking Fundamentals

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand the network path well enough to reason about latency, connection behavior, protocols, and failure.

## What You Need to Read / Learn

- IP addressing and routing at interview depth: packets move between hosts; you do not need router internals.
- DNS: name resolution, caching, TTL, and why DNS is part of the request path.
- TCP: connection-oriented, reliable byte stream, ordering, retransmission, handshake, and head-of-line behavior conceptually.
- UDP: datagram-oriented, lower protocol overhead, no built-in reliability or ordering.
- TLS: encryption in transit, certificate validation conceptually, handshake cost, and TLS termination.
- HTTP/HTTPS: request/response semantics, methods, status codes, headers, keep-alive, and HTTP connection reuse.
- Latency versus throughput: a system can have high throughput and still have poor tail latency.
- Timeouts: every network call can fail or hang; callers need bounded waiting.

## What You Need to Do

- [ ] Trace `https://example.com/feed` from browser to backend and back.
- [ ] Explain when TCP is preferable to UDP and vice versa.
- [ ] List where DNS, TLS, TCP, proxying, application work, and database work contribute latency.

## **Must Remember for the Interview**

- **Networks are unreliable: packets can be delayed, dropped, duplicated, or connections can fail.**
- **TCP provides ordered reliable transport; it does not make the application operation exactly-once.**
- **HTTPS is HTTP over TLS; TLS protects data in transit.**
- **DNS is cached and distributed, so changes are not always observed instantly.**
- **Every remote call needs a timeout strategy.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **DNS resolves names; TCP carries a reliable ordered byte stream; TLS encrypts/authenticates; HTTP defines application request/response semantics.**
- **Latency compounds across synchronous service hops.**
- **Persistent connections reduce repeated handshake cost.**
- **A network retry can create duplicate application requests.**
- **Timeouts are mandatory in distributed systems.**

## Self-Test Before Marking This Session Complete

- [ ] Can I trace a HTTPS request without notes?
- [ ] Can I explain TCP vs UDP in one minute?
- [ ] Can I explain why retries can produce duplicates even when TCP is reliable?
- [ ] Can I explain tail latency?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 3/46  
**Next:** Session 4
