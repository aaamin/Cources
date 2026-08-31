# Session 7 — Proxy, Reverse Proxy, API Gateway & Rate Limiting

## Outcome

You should be able to distinguish forward proxy, reverse proxy, load balancer, and API gateway; explain common gateway responsibilities; and choose/describe major rate-limiting algorithms and failure policies.

## Forward Proxy

A forward proxy represents the client side.

```text
Client → Forward Proxy → Internet/Service
```

Uses may include:
- corporate egress control;
- privacy;
- filtering;
- outbound routing.

The destination may see the proxy as the caller.

## Reverse Proxy

A reverse proxy represents the server side.

```text
Clients → Reverse Proxy → Backend servers
```

Common responsibilities:
- TLS termination;
- routing;
- compression;
- caching;
- hiding backend topology;
- request filtering.

A load balancer is often implemented with reverse-proxy behavior, but the concepts are not identical.

## API Gateway

An API gateway is an application/API-oriented entry layer.

```text
Clients
  ↓
API Gateway
  ├─ User Service
  ├─ Order Service
  └─ Search Service
```

Potential responsibilities:
- routing;
- authentication/authorization checks;
- rate limiting;
- request/response transformation;
- API version handling;
- aggregation;
- observability/correlation IDs.

### Avoid the “god gateway”

Do not move all business logic into the gateway. If every product rule lives there, it becomes a central bottleneck and tightly coupled monolith at the edge.

## Load Balancer vs Reverse Proxy vs API Gateway

| Concept | Primary purpose |
|---|---|
| Load balancer | distribute traffic across instances |
| Reverse proxy | server-side intermediary |
| API gateway | API entry/control/routing layer |
| Forward proxy | client-side intermediary |

One product can perform multiple roles. Interview answers should focus on responsibilities, not vendor labels.

## Service Discovery

Services need to know how to reach changing instances.

Conceptually:
- service registry;
- DNS/service naming;
- platform-provided discovery.

You only need recognition-level depth unless the prompt is specifically infrastructure discovery.

## Why Rate Limiting Exists

Rate limits protect:
- capacity;
- fairness;
- cost;
- external providers;
- abusive endpoints;
- per-tenant quotas.

Examples:
- 100 requests/minute per user;
- 1,000 writes/s per tenant;
- 10 login attempts/minute per IP;
- 10M API calls/month per customer.

## Fixed Window

Counter per fixed time interval.

```text
key:user123:12:00-12:01 → count
```

Advantages:
- simple;
- cheap.

Problem:
A user may send 100 requests at 12:00:59 and another 100 at 12:01:01, producing 200 requests in two seconds despite a nominal 100/minute limit.

## Sliding Window Log

Store timestamps of requests and count only those inside the last window.

Accurate but expensive:
- more storage;
- more operations.

## Sliding Window Counter

Approximate sliding behavior using neighboring fixed windows and weighting.

Less precise but more efficient.

## Token Bucket

Bucket has:
- capacity;
- refill rate.

A request consumes tokens.

Example:

```text
capacity = 100 tokens
refill = 10 tokens/sec
```

Allows a burst up to 100 while controlling sustained rate to ~10/sec.

Very common mental model for APIs.

## Leaky Bucket

Requests enter a bucket/queue and leave at a controlled rate.

Useful when you want smoother output.

Difference in intuition:

```text
Token bucket → permits bursts up to stored capacity
Leaky bucket → smooths output toward fixed drain rate
```

## Where State Lives

In a distributed system, all gateway replicas must agree enough on a user's quota.

Options:
- centralized/distributed cache such as Redis;
- partitioned rate-limit service;
- local approximate counters for some workloads.

Need atomic operations when concurrent gateways update the same counter.

## Per-User / Per-IP / Per-Tenant / Global

Different limits solve different problems.

Per-IP:
- helps unauthenticated abuse;
- can hurt users behind shared NAT.

Per-user:
- fair authenticated usage.

Per-tenant:
- prevents one customer from consuming all shared capacity.

Global:
- protects scarce dependency/capacity.

Often combine multiple layers.

## Failure Policy: Fail Open vs Fail Closed

If the rate-limit store is unavailable, what happens?

### Fail open
Allow requests.

Use when availability is more important than perfect enforcement.

Risk: overload/abuse.

### Fail closed
Reject requests.

Use when enforcement/cost/security is critical.

Risk: rate limiter outage becomes application outage.

Could also use:
- local fallback limits;
- cached quotas;
- degraded conservative mode.

State the policy based on endpoint importance.

## Rate-Limit Responses

For HTTP APIs:
- commonly 429 Too Many Requests;
- optionally include retry guidance/remaining quota.

Do not retry a 429 immediately in a tight loop.

## Multi-Region Challenge

A truly global quota is harder.

Options:
1. home-region enforcement;
2. globally coordinated state;
3. split quota per region;
4. approximate local quotas with periodic reconciliation.

Strong global limits increase coordination latency. Exactness vs availability is a trade-off.

## Worked Example — Login Endpoint

Requirements:
- stop brute force;
- avoid blocking entire offices sharing one public IP;
- protect authentication backend.

Possible design:
- per-IP burst limit;
- per-account limit;
- global backend protection;
- progressive delays/challenges.

If Redis limiter fails:
- perhaps conservative local fallback rather than globally failing all logins.

The correct policy depends on risk.

## Small Design Drills

1. Why might token bucket be preferred over fixed window for an API that permits short bursts?
2. A limiter DB is unavailable. When might fail-closed be appropriate?
3. Why can per-IP limiting harm legitimate users?
4. Why does a distributed rate limiter need atomic counters?
5. What is the core distinction between API gateway and load balancer?
6. Why is an exact cross-region global quota expensive?

<details>
<summary>Answer key</summary>

1. It allows controlled burst capacity while enforcing a sustained refill rate.
2. Security-sensitive or cost-critical action where exceeding quota is worse than temporary unavailability.
3. Many users can share one NAT/public IP.
4. Concurrent gateway updates can otherwise lose increments and undercount.
5. LB distributes across instances; gateway adds API-oriented routing/control/policy. Products may combine both.
6. Regions must coordinate shared state, adding latency and partition/availability trade-offs.

</details>

## Common Mistakes

- Treating forward and reverse proxies as the same direction.
- Making the gateway own business logic.
- Saying “Redis rate limiter” without algorithm/atomicity.
- Ignoring burst behavior.
- Using only per-IP limits for authenticated products.
- Failing open/closed without explaining why.
- Claiming an exact global quota is trivial.

## Must Remember

- **Forward proxy represents clients; reverse proxy represents servers.**
- **API gateway is an API control/entry layer, not the business layer.**
- **Fixed window is simple but has boundary bursts.**
- **Token bucket supports controlled bursts.**
- **Leaky bucket smooths output.**
- **Distributed limiting needs shared/partitioned state and atomic updates.**
- **Fail-open vs fail-closed is a business/reliability choice.**
- **Exact global quotas require coordination.**

## Interview Revision Summary

Rate limiter discussion:

```text
Who is limited?
What resource/action?
Limit + window/refill?
Burst allowed?
Algorithm?
Where is counter state?
Atomicity?
Multi-region?
What if limiter fails?
HTTP behavior?
```

## Explain Without Notes

Design a 100 requests/minute per-user API limit that allows a small burst, runs on ten gateway replicas, and behaves sensibly if the shared limiter store is down.

## Completion Checklist

- [ ] I distinguish proxy/reverse proxy/gateway/LB.
- [ ] I can explain fixed/sliding/token/leaky approaches.
- [ ] I understand distributed counter state.
- [ ] I can choose fail-open/fail-closed.
- [ ] I can discuss multi-region quota trade-offs.
