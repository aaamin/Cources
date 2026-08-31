# Session 26 — Observability, SLOs, Deployment, Migration, Security & Privacy

## 1. Must Learn

### Observability signals
- **Understand:** Know metrics, logs, traces, correlation IDs, dashboards, and alerts; use latency percentiles, error rate, throughput, saturation, queue depth, cache hit rate, dependency latency.
- **Decision/trade-off:** Visibility vs telemetry volume/cost/noise.

### SLI, SLO, SLA & error budget
- **Understand:** Understand measurement, reliability target, external commitment, and error-budget intuition.
- **Decision/trade-off:** Reliability investment vs delivery velocity/cost.

### Safe deployment & rollback
- **Understand:** Know rolling, canary, blue/green, health monitoring, and rollback.
- **Decision/trade-off:** Change speed/resource cost vs blast-radius reduction.

### Backward-compatible migration
- **Understand:** Understand expand-contract: add compatible schema → deploy compatible app → backfill → switch → remove old later.
- **Decision/trade-off:** Safe zero/low-downtime evolution vs temporary complexity.

### Backfills/reprocessing
- **Understand:** Know large migrations need throttling, resumability, observability, and correctness checks.
- **Decision/trade-off:** Migration speed vs production load/safety.

### Core security controls
- **Understand:** Understand authentication vs authorization, TLS, encryption at rest/in transit, secrets, least privilege, signed URLs, input validation, trust boundaries.
- **Decision/trade-off:** Security strength/operability vs complexity.

### Privacy & tenant isolation
- **Understand:** Know PII minimization, retention/deletion, audit logs, tenant isolation, and access boundaries.
- **Decision/trade-off:** Business/operational needs vs privacy obligations/risk.

### Abuse & production readiness
- **Understand:** Recognize rate limits, spam/scraping, WAF/DDoS awareness, capacity and cost as part of production design.
- **Decision/trade-off:** User openness/availability vs abuse protection/cost.

## 2. Should Know

- Use p50/p95/p99 instead of only averages for latency.
- Rollback planning must include schema compatibility, not just application binaries.
- Audit logs should protect sensitive actions without leaking unnecessary sensitive data.
- Security/privacy discussion should prioritize realistic risks, not enumerate every possible control.

## 3. Recognition Only

- SIEM
- Zero-trust architecture internals
- Advanced cryptography
- Dual-read/dual-write migration details

## 4. Important Comparisons

- Metrics vs logs vs traces.
- p50 vs p95 vs p99.
- SLI vs SLO vs SLA.
- Rolling vs canary vs blue/green deployment.
- Forward fix vs rollback.
- Expand-contract vs breaking schema change.
- Authentication vs authorization.
- Encryption in transit vs at rest.
- Reliability/security benefit vs operational/cost overhead.

## 5. Important Interview Questions

1. Which SLIs actually represent user experience?
2. What alert should wake someone up vs remain on a dashboard?
3. How do we deploy this change with limited blast radius?
4. Can the old and new application versions run against the same schema during rollout?
5. How do we backfill without overwhelming production?
6. What are the sensitive assets and trust boundaries?
7. What PII exists and how long should it be retained?
8. How is one tenant prevented from accessing another tenant's data?

## 6. Common Interview Mistakes

- **“We’ll add logging” as observability** → Choose signals tied to failures/SLOs.
- **Using average latency only** → Use tail percentiles.
- **Breaking schema then deploying app** → Use backward-compatible expand-contract sequence.
- **Huge unthrottled backfill** → Throttle, checkpoint, monitor, validate.
- **Authentication = authorization** → Verify identity and permissions separately.
- **“Encrypt everything” with no key/secret boundaries** → Explain where trust and access controls matter.
- **Security checklist dump** → Prioritize likely threats and sensitive data.
- **Ignoring privacy/deletion/tenant isolation** → Include data lifecycle and isolation.

## 7. Communication

### Important Vocabulary

metric, log, trace, correlation ID, p50, p95, p99, SLI, SLO, SLA, error budget, canary, blue/green, rollback, expand-contract, backfill, authentication, authorization, least privilege, PII, audit log, tenant isolation

### Useful Interview Phrases

- “I’d monitor user-facing latency/error SLIs plus saturation of the likely bottleneck.”
- “I’d canary this change to reduce blast radius and keep rollback available.”
- “The schema change must remain compatible with both old and new application versions during rollout.”
- “I’d minimize PII and enforce tenant isolation at every relevant access boundary.”

### Important Questions to Ask the Interviewer

- **Question:** “What availability/latency target defines success?”  
  **Why it matters:** Determines SLOs and resilience investment.
- **Question:** “Can we tolerate a brief maintenance window?”  
  **Why it matters:** Changes migration/deployment approach.
- **Question:** “What data is sensitive or regulated, and what is its retention requirement?”  
  **Why it matters:** Changes security/privacy/storage design.
- **Question:** “What is the acceptable blast radius for deployment failure?”  
  **Why it matters:** Changes rollout strategy.

## 8. ⭐ Must Remember

1. Observe user-visible outcomes and bottleneck saturation.
2. Tail latency matters.
3. SLI measures; SLO targets; SLA is an external commitment.
4. Deploy gradually when blast radius matters and keep rollback viable.
5. Use backward-compatible expand-contract migrations.
6. Backfills must be throttled, resumable, observable, and validated.
7. Authentication and authorization are different.
8. Minimize/protect PII and enforce tenant isolation.
9. Production readiness includes abuse and cost, not only availability.

## 9. Study Priority

- **Priority A — Must master first:** metrics/logs/traces, latency/error/saturation, SLI/SLO/SLA, safe deployment/rollback.
- **Priority B — Must still learn:** expand-contract/backfills, core security, privacy/tenant isolation.
- **Priority C — Lower priority:** dual-read/dual-write recognition, WAF/DDoS implementation details, advanced observability/security internals.

## 10. Revision Checklist

- [ ] Choose useful observability signals and tail-latency metrics.
- [ ] Explain SLI/SLO/SLA.
- [ ] Choose a rollout strategy and rollback plan.
- [ ] Walk through expand-contract + safe backfill.
- [ ] Cover authentication/authorization/encryption/secrets/least privilege.
- [ ] Cover PII, retention/deletion, audit, tenant isolation, abuse, and cost.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
