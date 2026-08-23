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


## Important interview ideas

> **Important:** Network calls are not function calls. They have latency, can time out, can partially succeed, and the caller may not know whether the remote operation completed.

### A practical latency model

When one service calls another, think about:

```text
queueing
+ connection/setup if needed
+ network round trip
+ remote processing
+ downstream calls
+ response transfer
```

The actual numbers vary, but the structure matters. A design with five sequential cross-region calls is inherently vulnerable to high tail latency even if each service is individually fast.

### TCP reliability does not make the application reliable

TCP gives an ordered byte stream while the connection exists. It does not tell your application whether a timed-out payment request was executed on the server. Consider:

```text
client sends ChargeCard
server charges card
response packet is lost
client times out
```

The network did not provide the client with the result. Retrying blindly could charge twice. This is why application-level idempotency is necessary even on reliable transports.

### Connection reuse matters at scale

Repeatedly establishing TCP/TLS connections wastes round trips and CPU. Services use connection pools and keep-alive. HTTP/2 can multiplex streams over a single connection, reducing the need for many parallel TCP connections.

In an interview, you usually do not need to choose an HTTP version, but you should understand why connection pooling, keep-alive, and persistent WebSockets exist.

### Regional distance is architecture

Physical distance creates unavoidable latency. If a user in Singapore synchronously calls a database in Virginia, no software optimization can remove the propagation delay. Global systems therefore use patterns such as:

- CDN/edge delivery for static data;
- regional application servers;
- regional read replicas;
- user “home regions” for writes;
- asynchronous cross-region replication;
- strong cross-region coordination only for data that truly requires it.

### Timeouts are ambiguous

A timeout means **the caller stopped waiting**. It does not necessarily mean the callee did nothing. This distinction is critical for writes. Read retries are often safe; write retries may require idempotency keys or operation IDs.

## Protocol decision table

| Need | Typical choice | Reason |
|---|---|---|
| ordinary API request/response | HTTP(S) | simple, ubiquitous |
| bidirectional realtime | WebSocket | persistent two-way channel |
| one-way server events | SSE | simple server→client stream |
| low-latency media | UDP/QUIC-based protocols | app/protocol controls timeliness/recovery |
| internal typed RPC | HTTP/gRPC-style RPC | schemas, tooling, streaming options |

The table is not a set of hard rules. Requirements decide.

## Worked scenario — payment timeout

A checkout service calls a payment provider with a 2-second timeout.

1. Checkout sends `charge(order123, $50, idem_key=abc)`.
2. Provider processes the charge after 1.8 seconds.
3. Network delay prevents the response from reaching Checkout before 2 seconds.
4. Checkout times out.

The correct conclusion is **unknown outcome**, not “payment failed.” Checkout should query by idempotency key or retry with the same key. The provider must return the existing charge rather than creating another.

This one scenario connects networking to reliability and business correctness.

## Interview questions and model answers

### Q1. “TCP vs UDP?”

TCP gives reliable ordered bytes and congestion control, making it natural for APIs and most database connections. UDP gives datagrams without delivery/order guarantees and is useful when the application values timeliness or wants custom recovery, as in media and some gaming. I would choose based on semantics, not simply say one is faster.

### Q2. “Why does a CDN reduce latency?”

It serves data from geographically closer edge locations, reducing round-trip distance and often avoiding requests to the origin entirely. It also reduces origin bandwidth and compute load.

### Q3. “What is TLS termination?”

A load balancer or gateway can complete the TLS connection and decrypt traffic before routing to internal services. That centralizes certificate management but means internal traffic has crossed a trust boundary; sensitive systems may also encrypt service-to-service traffic.

### Q4. “Why should every downstream call have a timeout?”

Without a timeout, a slow dependency can occupy threads, connections, and memory indefinitely. Timeouts bound resource usage and allow the caller to choose a retry, fallback, degraded response, or failure. They should fit an end-to-end latency budget.

## Common mistakes to avoid

- “UDP is always faster.”
- “TCP guarantees my payment happened exactly once.”
- Treating a timeout as proof of failure.
- Ignoring cross-region round trips.
- Creating a new connection for every request.
- Forgetting TLS/authentication on public traffic.
- Making many sequential remote calls in a latency-sensitive request.

## Short revision note

**Network mental model:** resolve name → establish/reuse connection → secure it → exchange protocol messages → handle timeout/ambiguity. Remote calls are slower and less certain than local calls.

## Topics to revise

- [ ] DNS and TTL
- [ ] TCP reliability/order
- [ ] UDP trade-offs
- [ ] TLS/HTTPS
- [ ] connection reuse/pooling
- [ ] round-trip latency
- [ ] cross-region latency
- [ ] timeout ambiguity

## Interview-ready synthesis

### A strong 60–90 second explanation

I think of a remote call as an unreliable, latency-bearing operation rather than a local function call. DNS finds the endpoint, the transport carries bytes, TLS protects them, and HTTP/RPC defines application semantics. I use connection reuse and timeouts, and I remember that a timeout produces an uncertain outcome for writes, which is why idempotency is an application-level concern.

### How this topic connects to the wider system

- Performance: round trips and sequential calls determine much of tail latency.
- Reliability: timeouts, resets, and ambiguous outcomes require retry/idempotency policy.
- Scalability: connection reuse/pooling prevents handshake and socket overhead from becoming the bottleneck.
- Global design: physical region distance sets a latency floor and motivates regional serving/caching.

### Revision flashcards with answers

**Does TCP mean the operation happened once?**  
No. It transports bytes reliably while connected; application retries can still duplicate business operations.

**What does TLS give?**  
Encryption in transit, integrity, and endpoint authentication.

**Why keep connections alive?**  
To avoid repeated handshake/setup cost and reduce CPU/socket churn.

**What does a timeout mean?**  
The caller stopped waiting; the server may have completed, failed, or still be processing.

**Why is cross-region write slower?**  
Strong coordination adds physical network round trips between distant regions.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

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
