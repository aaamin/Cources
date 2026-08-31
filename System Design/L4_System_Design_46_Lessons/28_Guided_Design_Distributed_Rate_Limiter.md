# Session 28 — Guided Design — Distributed Rate Limiter

## Interview Prompt

> Design a distributed API rate limiter used by many gateway/application instances. It must support per-user and per-tenant quotas and protect backend capacity.

Later requirement:
> Enforce quotas across multiple regions.

Before reading reference material, spend **35–45 minutes** designing.

---

# STOP — Attempt First

Your first design should answer:

- What exactly is being limited?
- Which algorithm?
- Where is counter/token state?
- How are updates atomic?
- What happens if limiter is down?
- What happens during a burst?
- How do multi-region quotas work?

---

# Reference Reasoning

## 1. Requirements

Functional:
- evaluate request against policy;
- per-user/per-tenant limits;
- return allow/deny;
- expose remaining/retry information optionally;
- configuration changes.

Non-functional:
- very low latency;
- high availability;
- limiter should not become bottleneck;
- reasonably accurate enforcement;
- protect downstream during overload.

Clarify:
- hard exact quota or approximate?
- burst allowance?
- time unit?
- multi-region exactness?
- fail-open/closed?

These answers change architecture dramatically.

## 2. Algorithm Choice

Example requirement:
- 100 requests/minute/user;
- burst up to 20.

Token bucket fits:

```text
capacity = 20 or larger burst budget
refill = 100/min sustained equivalent
```

State per key:
```text
tokens
last_refill_time
```

Atomic operation:
1. compute refill since last update;
2. cap at capacity;
3. if token available, decrement and allow;
4. persist new state.

A fixed window is simpler if boundary burst is acceptable.

## 3. Key Structure

```text
user:{user_id}:{policy_id}
tenant:{tenant_id}:{policy_id}
```

A request may need multiple checks:
- user;
- tenant;
- IP;
- global dependency capacity.

Need decide atomicity across multiple quota dimensions. Often independently evaluate and deny if any exceeded.

## 4. Architecture

```text
Client
  ↓
Gateway
  ↓
Rate Limit Library/Service
  ↓
Partitioned In-Memory Counter Store
```

Two common patterns:

### Gateway-integrated
Gateway calls shared store atomically.

Pros:
- one less service hop.

### Dedicated rate-limit service
Central policy logic, partitioned/scaled.

Pros:
- consistent behavior/config;
- easier evolution.

Cost:
- extra network hop.

Choose based on organization/scale.

## 5. Atomicity

Two gateways simultaneously see 1 token.

Without atomic update:
```text
both read token=1
both allow
both write 0
```

Quota exceeded.

Use:
- atomic increment/decrement;
- Lua/server-side transaction;
- compare-and-set;
- dedicated counter primitive.

Do not lock globally.

## 6. Partitioning

Partition limiter state by hash of key.

```text
hash(user/tenant) → limiter shard
```

Benefits:
- distribute state/traffic.

Hot key:
- one huge tenant/global key can overload one shard.

Mitigations:
- dedicated shard;
- hierarchical/local token allocation;
- split tenant subkeys with approximate aggregation;
- isolate very large customer.

## 7. Local Token Leasing

For extreme scale, global store can allocate chunks to gateways.

Example:
```text
global quota 10k/s
gateway A gets 500 local tokens
gateway B gets 500
...
```

Gateways enforce locally until lease exhausted.

Advantages:
- fewer central calls;
- lower latency.

Cost:
- quota becomes approximate;
- unused leases waste capacity;
- redistribution delay.

Useful when exactness is not critical.

## 8. Fail-Open vs Fail-Closed

### Login brute-force limiter
Security matters → fail-closed/conservative local fallback may be appropriate.

### Noncritical recommendation API
Availability may matter more → fail-open with backend global overload protection.

### Expensive third-party API
Fail-closed protects money/provider quota.

State policy explicitly.

## 9. Config Distribution

Policies:
```text
plan A: 100/min
plan B: 1000/min
```

Store authoritative configuration separately and distribute/cache it to limiter instances.

Config changes should have version/effective time.

Do not read policy DB for every request.

## 10. Response

If denied:
```http
429 Too Many Requests
```

May include:
- Retry-After;
- remaining quota;
- policy info.

Clients should back off rather than immediate retry.

## 11. Multi-Region — Exact Global Quota

Requirement:
> User gets exactly 100 requests/minute globally across Singapore + Frankfurt.

If each region independently allows 100:
- total can become 200.

Exact global enforcement needs coordination/shared authoritative state, which adds cross-region latency and partition availability problems.

Options:

### A. Home region
Each quota key has an authoritative region.
Remote request forwards limiter check.

Strong but cross-region latency.

### B. Global strongly consistent store
Simplifies semantics, expensive coordination.

### C. Split quota
Assign:
```text
Singapore 60
Frankfurt 40
```

Fast/available but capacity can be stranded.

### D. Token leasing
Global coordinator periodically allocates budgets to regions.

Approximate but practical.

### E. Eventually reconciled counters
Suitable for billing/soft quotas, not strict backend protection.

Interview answer depends on “exact vs approximate.”

## 12. Protecting Backend Capacity

Per-user quotas are not enough.

If 10M users each stay below quota, backend can still overload.

Add:
- global/service admission limit;
- concurrency limit;
- load shedding.

Rate limiter is one part of overload protection.

## 13. Failure Scenarios

### Counter store slow
Limiter can cause every API request to slow.
Use:
- tiny timeout;
- local fallback;
- circuit breaker;
- fail policy.

### Counter store partition lost
Only subset of users affected if sharded.
Need replication/failover.

### Config service down
Use cached last-known-good policies.

### Clock problem
Token refill based on timestamps; use server/store time/monotonic interval semantics where possible, not untrusted client clock.

## 14. Abuse Concerns

Keys should come from authenticated identity/server-derived IP, not arbitrary client-provided header.

Prevent high-cardinality attack:
- attacker creates millions of fake limiter keys;
- validate/authenticate before expensive state where possible;
- TTL old keys;
- IP/global front-door protection.

## 15. Trade-Off Matrix

| Design | Accuracy | Latency | Availability |
|---|---|---|---|
| Shared central check | high | higher | central dependency |
| Regional independent | low global accuracy | low | high |
| Split quota | bounded/approx | low | high |
| Token lease | approximate | very low | good |
| Global consistent | high | highest cross-region | partition trade-off |

## Interviewer Questions

1. Why not fixed window?
2. What if Redis is down?
3. How do you prevent race in token decrement?
4. What if one tenant gets 50× traffic?
5. Can rate limiter itself become DDoS target?
6. How would you enforce a monthly quota?
7. What if global quota must be exact?

## Common Mistakes

- No algorithm.
- No atomicity.
- “Use Redis” as full answer.
- Exact global quota with no coordination.
- Fail-open/closed unexplained.
- User quotas but no global backend protection.
- Counter store on every request with no failure plan.
- Client-controlled identity.
- Ignoring hot tenants/high-cardinality abuse.

## Must Remember

- **Define exactness and burst semantics first.**
- **Token bucket is a strong API default when bursts are allowed.**
- **Distributed updates must be atomic.**
- **Rate-limiter latency is on the request path.**
- **Limiter failure policy is endpoint-specific.**
- **Exact global quota requires coordination.**
- **Approximate regional/token-leased quotas often scale better.**
- **Per-user limits do not replace global overload protection.**

## Self-Score

Use the 40-point rubric and record the two weakest categories.
