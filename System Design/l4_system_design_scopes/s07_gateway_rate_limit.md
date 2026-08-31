# Session 7 — Proxy, API Gateway & Rate Limiting

## 1. Must Learn

### Forward vs reverse proxy
- **Understand:** Understand whose traffic each proxy represents and basic routing/security roles.
- **Decision/trade-off:** Client-side mediation vs server-side ingress control.

### API gateway
- **Understand:** Know centralized routing plus cross-cutting concerns such as authentication, aggregation, and policy.
- **Decision/trade-off:** Centralized control/convenience vs added hop, coupling, and failure surface.

### TLS termination & edge auth
- **Understand:** Understand terminating secure client traffic and optionally validating credentials near ingress.
- **Decision/trade-off:** Centralization vs preserving end-to-end trust boundaries.

### Rate limiting purpose & scope
- **Understand:** Know limits protect capacity, fairness, cost, and abuse controls; choose per-user/IP/tenant/global scope.
- **Decision/trade-off:** Fairness/protection vs false rejection and state/coordination cost.

### Rate-limit algorithms
- **Understand:** Understand fixed window, sliding-window intuition, token bucket, and leaky bucket.
- **Decision/trade-off:** Implementation simplicity vs burst tolerance vs smooth output.

### Distributed enforcement & failure policy
- **Understand:** Know counters may need shared/distributed state and define fail-open vs fail-closed behavior.
- **Decision/trade-off:** Availability vs strict quota/security enforcement.

## 2. Should Know

- Service discovery conceptually behind gateways/reverse proxies.
- Rate-limit response behavior: clear status/retry metadata.
- Global limits are harder than regional/local limits because coordination is expensive.

## 3. Recognition Only

- Sliding-window log vs counter variants
- Hierarchical rate limits

## 4. Important Comparisons

- Load balancer vs reverse proxy vs API gateway vs rate limiter.
- Fixed window vs sliding window vs token bucket vs leaky bucket.
- Per-user vs per-IP vs per-tenant vs global limits.
- Fail-open vs fail-closed rate limiting.

## 5. Important Interview Questions

1. Where should rate limiting happen?
2. Which identity should the limit apply to?
3. Do clients need short bursts?
4. What happens when the rate-limit store is unavailable?
5. Why is a global multi-region quota harder than a regional quota?
6. What belongs in the gateway vs application service?

## 6. Common Interview Mistakes

- **Treating LB and gateway as identical** → Explain their distinct responsibilities even if one product can implement both.
- **Per-IP limiting for authenticated fairness** → Prefer user/tenant identity when available; IPs can be shared or rotated.
- **No failure policy** → State fail-open/fail-closed based on risk.
- **Ignoring burst semantics** → Choose an algorithm that matches desired behavior.
- **Putting all business logic in gateway** → Keep gateway focused on cross-cutting ingress concerns.

## 7. Communication

### Important Vocabulary

forward proxy, reverse proxy, API gateway, routing, TLS termination, rate limit, fixed window, sliding window, token bucket, leaky bucket, quota, fail-open, fail-closed

### Useful Interview Phrases

- “I’d rate-limit on the strongest identity available, such as tenant or user.”
- “Token bucket allows controlled bursts while enforcing a long-term rate.”
- “The failure policy depends on whether availability or strict enforcement is more important.”

### Important Questions to Ask the Interviewer

- **Question:** “Should legitimate users be allowed short bursts?”  
  **Why it matters:** Changes rate-limit algorithm.
- **Question:** “Is this limit regional or truly global?”  
  **Why it matters:** Determines coordination complexity.
- **Question:** “What is worse: allowing excess traffic or rejecting valid traffic?”  
  **Why it matters:** Determines fail-open vs fail-closed.

## 8. ⭐ Must Remember

1. Gateway, reverse proxy, load balancer, and rate limiter are distinct concepts.
2. Rate limits protect fairness and capacity, not just abuse.
3. Choose limit identity and scope explicitly.
4. Token bucket is a common burst-friendly choice.
5. Global enforcement requires more coordination.
6. Always define failure behavior.

## 9. Study Priority

1. Study first: proxy/gateway roles.
2. Study next: rate-limit purpose, scope, and algorithms.
3. Finish with: distributed counters and fail-open/fail-closed.

## 10. Revision Checklist

- [ ] Differentiate LB, reverse proxy, gateway, and limiter.
- [ ] Choose a rate-limit algorithm and identity.
- [ ] Discuss distributed/global enforcement.
- [ ] Define rate-limit response behavior.
- [ ] Explain fail-open vs fail-closed.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
