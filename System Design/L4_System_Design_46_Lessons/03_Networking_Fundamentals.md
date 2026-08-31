# Session 3 — Networking Fundamentals

## Outcome

You should understand enough networking to trace a request from a client to a backend, explain where latency and failure can occur, and make sound system-design choices involving DNS, TCP/UDP, TLS, HTTP, connection reuse, bandwidth, and timeouts.

## Why This Matters

System-design diagrams often hide the network behind arrows. In reality, every distributed call adds:

- DNS/routing work;
- connection establishment or reuse;
- network latency;
- serialization;
- queueing;
- timeout/failure risk.

Microservices, databases, caches, message brokers, CDNs, and gateways all communicate over networks. You do not need to become a network engineer, but you must understand what each arrow costs.

## Mental Model — One HTTPS Request

```text
User enters URL
    ↓
DNS resolves hostname → IP
    ↓
Client opens/reuses transport connection
    ↓
TLS establishes encrypted session
    ↓
HTTP request travels to server/LB/CDN
    ↓
Backend processing
    ↓
HTTP response returns
```

Modern protocols may optimize parts of this flow, but this model is sufficient for interview reasoning.

## IP Addresses

An IP address identifies a network endpoint/routing destination.

You typically work with hostnames rather than hard-coded IPs because:

- infrastructure changes;
- load balancers have multiple addresses;
- global systems route users differently;
- failover can change destinations.

Do not confuse DNS with a load balancer. DNS maps names to routing information; a load balancer distributes traffic after requests reach the appropriate endpoint.

## DNS

DNS translates a hostname such as:

```text
api.example.com
```

into an address the client can route to.

Important concepts:

- DNS records are cached.
- TTL controls how long cached answers may remain.
- DNS-based traffic steering can route users by geography/health.
- DNS changes are not necessarily instantaneous because caches exist.
- DNS is usually part of global routing, not request-by-request application balancing.

### Failure implication

If a region fails and DNS is used for failover, previously cached records can delay movement of some clients.

## TCP

TCP provides a reliable ordered byte stream.

Conceptually it handles:

- connection establishment;
- retransmission;
- ordering;
- congestion control;
- detecting broken connections imperfectly over time.

Most traditional HTTP traffic uses TCP, although HTTP/3 uses QUIC over UDP.

The key interview point: establishing new connections costs latency. Reusing connections is valuable.

## UDP

UDP is connectionless and does not itself guarantee delivery or ordering.

It can be useful when:

- application/protocol handles reliability differently;
- latency matters more than retransmitting every packet;
- protocols such as QUIC build their own semantics over UDP.

For most application system-design interviews, you need recognition-level understanding rather than implementing transport reliability.

## TLS

TLS provides encryption in transit and server authentication.

Typical path:

```text
TCP/transport established
    ↓
TLS handshake
    ↓
Encrypted application traffic
```

TLS termination may occur at:

- CDN;
- load balancer;
- reverse proxy/API gateway;
- application server.

If traffic from the load balancer to services also needs encryption, TLS may continue internally.

## HTTP / HTTPS

HTTP provides application request/response semantics.

Important concepts:

- methods such as GET/POST/PUT/PATCH/DELETE;
- headers;
- status codes;
- request body;
- response body;
- keep-alive/connection reuse;
- caching behavior;
- idempotency semantics.

HTTPS means HTTP protected by TLS.

## Keep-Alive and Connection Reuse

Without reuse, every request may repeatedly pay connection/TLS setup cost.

Persistent pooled connections reduce:

- latency;
- CPU handshake cost;
- connection churn.

This matters between:
- clients and gateways;
- services and databases;
- services and downstream APIs.

## Connection Pools

A service rarely opens a brand-new DB connection for every query. It borrows one from a pool.

If a pool has 50 usable connections and 500 requests simultaneously need a database call, not all 500 run immediately. Some wait for a connection; if waiting exceeds limits, requests time out or fail.

This creates queueing even when CPU looks low.

Connection pools should be bounded. Unlimited connections can overwhelm the database.

## Latency vs Throughput

### Latency
Time for one operation to complete.

### Throughput
Amount of work completed per unit time.

A system can have:
- low latency but low total throughput;
- high throughput but poor p99 latency.

Do not use the words interchangeably.

## Bandwidth

Bandwidth is how many bits/bytes can traverse a link per second.

A server may have enough CPU to handle 10k metadata requests/s but still fail to serve large videos because network bandwidth is exhausted.

This is why media architectures separate:
- metadata/API serving;
- object/media delivery through CDN/object storage.

## Timeouts

Every remote call should have a bounded waiting time.

Without a timeout:

```text
Dependency becomes slow
    ↓
Requests wait
    ↓
Threads/tasks/connections accumulate
    ↓
Resource exhaustion
    ↓
Whole service degrades
```

Timeout selection is a trade-off:
- too short → healthy slow requests fail;
- too long → resources remain occupied and failures propagate.

Later reliability lessons add retries, jitter, circuit breakers, and backpressure.

## Synchronous Call Chains

Consider:

```text
Client → API → Service A → Service B → Service C → DB
```

Each dependency contributes latency and failure probability.

If each downstream call takes ~50 ms, the end-to-end request does not magically remain 50 ms. Queueing and serialized calls add up.

Long synchronous chains are one reason microservices can harm latency/reliability if decomposed poorly.

## Regional Latency

Distance matters because network propagation takes time.

A Bangladesh client talking to a nearby Asia region generally has lower network latency than talking across continents. Global systems therefore use:

- regional deployment;
- CDN/edge caching;
- home-region data;
- geo routing.

But putting everything in multiple regions introduces data consistency and operational complexity.

## Worked Example — API Appears Slow but CPU Is Low

Symptoms:
- API p95 = 2 seconds;
- CPU = 20%;
- memory fine.

Possible causes:
- DB connection pool exhausted;
- slow downstream API;
- DNS/connect/TLS churn;
- database lock wait;
- network packet loss/retransmission;
- queueing at load balancer;
- thread/task pool blocking;
- remote timeout set too high.

The lesson: low CPU does not mean the server is healthy.

## Small Design Drills

1. Why does keep-alive improve latency?
2. Why can DNS failover take time?
3. A server uses only 15% CPU but requests queue for 3 seconds. Name three network/dependency causes.
4. Why are connection pools bounded?
5. Does TCP guarantee your business request executes only once?
6. Why should media usually not flow through the application server when object storage/CDN can serve it directly?

<details>
<summary>Answer key</summary>

1. It avoids repeated connection and often TLS setup.
2. Clients/resolvers cache DNS records according to TTL and behavior.
3. Pool exhaustion, slow downstream service, network latency/loss, handshake churn, load-balancer queueing, etc.
4. Databases/downstreams have finite connection capacity; unbounded pools shift overload downstream.
5. No. Transport reliability is not business-level exactly-once execution; retries/disconnections can create duplicate requests.
6. It wastes application-server bandwidth/CPU and prevents efficient edge delivery.

</details>

## Common Mistakes

- Saying DNS load-balances every request in the same way an L7 LB does.
- Assuming low CPU means no bottleneck.
- Ignoring connection pools.
- Forgetting timeout behavior.
- Treating TCP reliability as application exactly-once.
- Confusing bandwidth and latency.
- Sending huge files through application servers unnecessarily.
- Creating long synchronous service chains.

## Must Remember

- **Every distributed arrow is a network call with latency and failure risk.**
- **DNS is cached and failover is not always instantaneous.**
- **TCP gives reliable ordered bytes, not business-level exactly-once.**
- **TLS protects data in transit.**
- **Connection reuse reduces setup cost.**
- **Connection pools create bounded downstream concurrency.**
- **Latency and throughput are different.**
- **Low CPU can coexist with severe dependency or network bottlenecks.**
- **Remote calls need timeouts.**

## Interview Revision Summary

Trace:

```text
DNS → transport → TLS → HTTP → LB/gateway → app → dependency → response
```

At each arrow ask:
- new or reused connection?
- latency?
- timeout?
- pool?
- bandwidth?
- what if it is slow?
- what if it fails?

## Explain Without Notes

Explain why an API server with low CPU could still have 2-second p99 latency. Include connection pools, downstream calls, timeouts, and network latency.

## Completion Checklist

- [ ] I can trace an HTTPS request.
- [ ] I can explain DNS, TCP, UDP, TLS, and HTTP at interview depth.
- [ ] I understand keep-alive and pools.
- [ ] I distinguish latency, throughput, and bandwidth.
- [ ] I can reason about slow remote dependencies.
