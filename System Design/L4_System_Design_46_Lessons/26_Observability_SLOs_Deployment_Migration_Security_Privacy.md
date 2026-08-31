# Session 26 — Observability, SLOs, Deployment, Migration, Security & Privacy

## Outcome

You should be able to perform a concise production-readiness review covering metrics/logs/traces, SLI/SLO/SLA/error budgets, deployment and rollback, backward-compatible schema changes/backfills, authentication/authorization/encryption/secrets/least privilege, WAF/DDoS awareness, PII/retention/deletion/audit, abuse prevention, tenant isolation, and cost.

This session is broad by design. You do not need security/SRE specialist depth; you need to recognize what matters in an L4 architecture.

# Part A — Observability

Observability answers:
> What is the system doing, and why?

## Metrics

Numeric time-series.

Examples:
- RPS;
- error rate;
- p95/p99 latency;
- CPU/memory;
- DB connection pool usage;
- cache hit rate;
- queue depth;
- oldest queue message age;
- consumer lag;
- saturation.

Metrics are good for dashboards/alerts/trends.

## Logs

Discrete events with context.

Good structured log:
```json
{
  "level":"error",
  "requestId":"r123",
  "orderId":"o55",
  "service":"payment",
  "errorCode":"PROVIDER_TIMEOUT"
}
```

Avoid:
- passwords/tokens;
- full sensitive PII;
- unbounded noisy logs.

## Traces

Follow request across services:

```text
Gateway 20ms
 ↓
Order 50ms
 ↓
Payment 800ms
 ↓
DB 20ms
```

Trace reveals where latency occurs.

## Correlation IDs

Shared request/trace identifier lets logs across services be connected.

Useful even without sophisticated tracing.

## Percentiles

Average latency hides tail.

Example:
```text
p50 = 100 ms
p95 = 500 ms
p99 = 4 sec
```

1% of users can experience severe latency while average looks fine.

For distributed fan-out, tails compound because request waits for slow subcalls.

## Saturation

Measure resource nearing limit:
- CPU;
- connection pool;
- thread pool;
- DB max connections;
- disk;
- queue backlog;
- provider quota.

A service can have low CPU but saturated DB pool.

# Part B — SLI, SLO, SLA

## SLI
Measured indicator.

Examples:
```text
successful requests / total
p99 latency
durable message delivery
```

## SLO
Internal/target objective.

Example:
```text
99.9% successful requests per month
```

## SLA
External contractual commitment with business/legal consequences.

Do not casually call every uptime target an SLA.

## Error Budget

If SLO is 99.9%, allowable unavailability/error fraction is roughly 0.1% for the measurement window.

Teams can use remaining error budget to balance reliability vs feature velocity.

Recognition-level is enough.

# Part C — Alerts

Alert on user impact or approaching failure:
- sustained error rate;
- latency;
- queue age;
- replication lag;
- disk;
- cache hit collapse;
- failed backups;
- region health.

Avoid alerting on every single error.

Good alert should be actionable.

# Part D — Deployment Strategies

## Rolling

Replace instances gradually.

Pros:
- efficient;
- common.

Need:
- backward compatibility between old/new versions during overlap;
- connection draining;
- rollback.

## Blue/Green

Two environments:
```text
Blue = current
Green = new
```

Switch traffic.

Pros:
- fast rollback/switch.

Costs:
- duplicate environment/capacity;
- data compatibility still matters.

## Canary

Send small percentage to new version.

Monitor:
- errors;
- latency;
- business correctness.

Then increase traffic.

Canary reduces blast radius but does not detect rare/data-specific bugs automatically.

# Part E — Backward-Compatible Schema Change

Dangerous:

```text
deploy code requiring new column
then alter DB later
```

or:
```text
drop old column while old app still reads it
```

Use expand-contract.

## Expand-Contract

Example rename `name` → `display_name`.

### Expand
Add new field/table/index while old remains.

### Compatible app
Deploy version that can handle both.

### Backfill
Populate existing rows.

### Switch
Move reads/writes to new model.

### Contract
After all old code/data dependencies disappear, remove old field.

This supports rolling deploys and rollback.

## Backfills

Large backfill can overload production DB.

Use:
- batches;
- checkpoints;
- throttling;
- retries;
- observability;
- idempotent updates;
- pause/resume.

Do not lock/rewrite a billion-row table aggressively during peak unless system supports it.

## Dual Read / Dual Write Recognition

Migration may temporarily:
- write old + new;
- read fallback from old.

Risks:
- divergence;
- partial write failure.

Need monitoring/reconciliation.

Prefer DB-level compatible change or outbox/CDC where appropriate.

# Part F — Security

## Authentication
Who are you?

## Authorization
Are you allowed to perform action?

Always check authorization on server, not trust client UI.

## TLS
Encrypt data in transit.

## Encryption at Rest
Protect stored data/media/backups.

Need key management/rotation policies at platform level.

## Secrets
Passwords/API keys/private keys should not live in:
- source code;
- logs;
- client bundle.

Use secret-management systems/environment injection with access controls.

## Least Privilege

Each service/user gets minimum permission needed.

Example:
Notification worker should not have permission to modify payment ledger.

Reduces blast radius.

## Signed URLs

Authorize time-limited direct object access.

## Input Validation

Validate:
- type/format;
- size;
- allowed values;
- ownership;
- file uploads.

Protect DB/API from malformed/abusive input.

## WAF Awareness

Web Application Firewall can help block/filter common malicious patterns and abusive traffic at edge.

Do not treat WAF as replacement for secure code/auth/rate limits.

## DDoS Awareness

Layer defenses:
- CDN/edge;
- managed DDoS protection;
- rate limiting;
- autoscaling;
- load shedding;
- origin protection.

A malicious flood should not have direct unlimited access to the origin.

# Part G — Privacy

## PII

Examples:
- name;
- email;
- phone;
- location;
- identifiers.

Know what you collect and why.

Reduce:
- storage;
- logs;
- replication;
- retention.

## Retention and Deletion

If user deletes account:
- primary DB;
- caches;
- search index;
- derived views;
- object storage;
- backups according to policy/legal constraints.

Deletion in distributed systems is a workflow.

## Audit Logs

For sensitive changes:
```text
who
what
when
resource
result
```

Protect audit log from ordinary mutation and avoid storing secrets.

## Tenant Isolation

Multi-tenant SaaS must prevent tenant A accessing B.

Layers:
- authorization;
- tenant key in data access;
- partition/schema/account isolation as needed;
- cache keys include tenant;
- rate quotas.

A missing tenant dimension in cache key can become a data leak.

# Part H — Abuse

Think about:
- spam;
- scraping;
- credential stuffing;
- bot signups;
- upload abuse;
- expensive query abuse;
- webhook abuse.

Controls:
- rate limits;
- quotas;
- reputation/challenges;
- input size caps;
- moderation;
- provider isolation.

# Part I — Production Readiness Review

At the end of a design, do not mechanically list every topic. Prioritize what matters.

For a chat system:
- connection metrics;
- message durability;
- queue lag;
- auth;
- abuse/spam;
- multi-region.

For a ticket system:
- inventory invariant;
- payment reconciliation;
- audit;
- DB failover;
- rate limiting/waiting room.

## Worked Example — Deploying New Order Schema

Goal:
replace `shipping_address_text` with normalized `address_id`.

Safe migration:
1. add `address_id` nullable;
2. deploy app that writes both old/new where possible;
3. backfill historical orders in batches;
4. compare counts/errors;
5. switch reads to new with fallback;
6. monitor;
7. stop old writes;
8. after rollback window, remove old field.

During backfill:
- throttle DB;
- checkpoint;
- idempotent;
- monitor p99/replication lag.

## Small Design Drills

1. Why can average latency be misleading?
2. SLO vs SLA?
3. Why must rolling deployment keep old/new schema compatibility?
4. What is expand-contract?
5. Why is a large backfill operationally dangerous?
6. Why is WAF not enough security?
7. Give two tenant-isolation failures caused by caching/data access.
8. Why should backup restore be monitored/tested?

<details>
<summary>Answer key</summary>

1. Tail users can suffer severely while average remains low; use p95/p99.
2. SLO is reliability objective; SLA is external contractual commitment.
3. Old and new versions run simultaneously and may share data.
4. Add compatible new structure, migrate/backfill/switch, remove old only later.
5. Can saturate DB, locks/IO/replication and impact live traffic.
6. Auth, authorization, secure code, secrets, least privilege, validation etc. are still required.
7. Cache key missing tenant; query missing tenant filter/authorization.
8. A backup that cannot be restored within RTO is not useful DR.

</details>

## Common Interview Mistakes

- “We add monitoring” with no metrics.
- Average latency only.
- Alert every error.
- Calling SLO an SLA.
- Schema change with no overlap/backward compatibility.
- Massive unthrottled backfill.
- Security laundry list unrelated to system.
- JWT/authentication with no authorization.
- Secrets in logs/config repo.
- Missing tenant in cache key.
- User deletion only from primary table.
- DDoS solved only by autoscaling.
- No audit/reconciliation for sensitive operations.

## Must Remember

- **Observe metrics, logs, and traces.**
- **Tail latency p95/p99 matters.**
- **Measure saturation, not CPU alone.**
- **SLI measures; SLO targets; SLA contracts.**
- **Rolling deploys require backward compatibility.**
- **Use expand-contract for risky schema evolution.**
- **Backfills must be batched, throttled, resumable, and observable.**
- **Authentication and authorization are different.**
- **Use TLS, encryption, secrets management, least privilege.**
- **WAF/DDoS protection complements—not replaces—secure design.**
- **PII retention/deletion applies to derived systems too.**
- **Tenant isolation must exist in auth, storage, cache, and quotas.**

## Interview Revision Summary

Production review:

```text
Metrics/logs/traces?
p95/p99?
SLO?
Failure alerts?
Deployment/rollback?
Schema compatibility?
Backfill?
Authn/authz?
Encryption/secrets?
Least privilege?
Input/rate limits?
WAF/DDoS?
PII?
Retention/deletion?
Audit?
Tenant isolation?
Abuse?
Cost?
```

## Explain Without Notes

Give a three-minute production-readiness review for an e-commerce checkout system. Include observability, deployment/schema migration, security, tenant/user data, abuse, and payment audit/reconciliation.

## Completion Checklist

- [ ] I know metrics/logs/traces and useful system metrics.
- [ ] I understand SLI/SLO/SLA/error budget.
- [ ] I compare rolling/canary/blue-green.
- [ ] I can design expand-contract + backfill.
- [ ] I understand core security controls.
- [ ] I consider privacy/tenant isolation/abuse.
- [ ] I can give a concise production-readiness review.
