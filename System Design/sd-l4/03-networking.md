# Lesson 03 — Networking Fundamentals

**Phase:** Fundamentals  
**Session:** 3/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn the networking concepts that repeatedly appear in system design: DNS, TCP, UDP, TLS, HTTP, connection reuse, latency, and timeouts. L4 depth means understanding behavior and trade-offs, not packet-level implementation.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## A typical HTTPS request

A request may involve DNS resolution, a TCP/QUIC connection, TLS establishment, HTTP, a load balancer, application processing, and storage. Each stage adds latency and can fail. DNS and connections are often cached/reused, so not every request pays every setup cost.

## DNS

DNS maps names such as `api.example.com` to addresses. It is hierarchical and heavily cached. In design, DNS can participate in global routing, failover, and CDN selection. Changes are not instantaneous because resolvers obey TTLs.

## TCP

TCP provides a reliable ordered byte stream. It handles loss and retransmission so applications see ordered bytes. Reliability adds connection state and latency. For system design, understand why persistent connections help and why connection failure requires recovery.

## UDP

UDP sends independent datagrams without delivery or ordering guarantees. It is useful when an application or higher-level protocol wants different reliability behavior, such as media, gaming, DNS, or QUIC-based transport. Do not reduce the comparison to “UDP is faster.”

## TLS and HTTPS

TLS provides encryption in transit, server authentication, and integrity. TLS may terminate at an edge proxy or load balancer; that defines a trust boundary. Internal traffic still needs appropriate protection based on threat model.

## Latency, throughput, timeouts

Latency is time per operation; throughput is work per unit time. A system can have high throughput with poor tail latency. Timeouts bound remote calls so slow dependencies do not hold resources forever and trigger cascades.

## Worked example — one HTTPS API call

A phone requests `https://api.example.com/profile`: DNS finds an endpoint; the client opens or reuses a transport connection; TLS secures it; HTTP carries the request; a load balancer chooses an app server; the app may call cache/database; the response returns. Delay can come from DNS, handshakes, network RTT, queueing, application logic, and data access.

## Interview lens

Networking appears indirectly in questions about CDN, WebSockets, timeouts, gateways, and global routing. Explain only the networking behavior relevant to the design.

## What to remember

DNS finds a destination; transport carries data; TLS protects it; HTTP defines application request/response semantics.

## Review after reading

1. What does DNS do?
2. What guarantee does TCP provide?
3. Why reuse connections?
4. What does TLS protect?
5. Why must remote calls have timeouts?

## Deeper study notes

### Round trips are a design currency

Network latency is often dominated by round trips, especially across regions. A service that performs five sequential remote calls can be much slower than one that performs the same work in parallel or locally. When you see a long synchronous chain, count the network hops and ask whether some can be removed, parallelized, cached, or made asynchronous.

### Connection establishment vs reuse

Opening a fresh connection may require DNS resolution, a transport handshake, and TLS negotiation. Reusing connections avoids much of that overhead. This is why HTTP clients, database clients, and service meshes maintain connection pools. At scale, poor reuse can consume CPU and file descriptors even before business logic becomes the bottleneck.

### HTTP versions at interview depth

You do not need protocol trivia, but a useful mental model is: HTTP/1.1 commonly reuses connections but has limitations around request concurrency; HTTP/2 multiplexes many streams over one connection; HTTP/3 runs over QUIC and avoids some TCP-level head-of-line behavior. Mention these only when the prompt makes transport behavior relevant.

### Cross-region latency

The speed of light creates a floor. A user in Asia calling a database only in North America cannot get single-digit-millisecond latency. Global systems therefore use regional edges, caches, replicas, or home-region strategies. Strong synchronous cross-region writes pay additional round trips and should be justified by correctness requirements.

### Failure is not only packet loss

From an application perspective, network failure includes DNS failure, connection refusal, handshake failure, timeout, reset, partial response, and a response that arrives after the caller has already timed out. The caller may not know whether the remote side completed the operation, which is why idempotency becomes important for writes.

### Common mistakes

- Saying UDP is simply “faster” without mentioning missing guarantees.
- Assuming a timeout means the remote operation did not happen.
- Forgetting TLS when discussing public traffic.
- Treating network latency as negligible between regions.
- Creating one new connection per request in a high-throughput design.

### Recall model

```text
Name resolution → connection → security → application protocol → service
       DNS           TCP/QUIC       TLS              HTTP
```


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

**Next:** Lesson 04 — Client–Server Architecture & Request Lifecycle
