# Lesson 26 — Observability, SLOs, Deployment, Security & Privacy

**Phase:** Fundamentals  
**Session:** 26/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn the production-readiness concerns that make a system operable: metrics/logs/traces, tail latency, SLOs, safe deployment, authentication/authorization, encryption, privacy, and abuse controls.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Metrics, logs, traces

Metrics answer aggregate questions such as QPS, error rate, p99, queue depth, cache hit rate. Logs record events with context. Traces connect one request across services so you can see where time/errors occurred. Correlation IDs tie telemetry together.

## Tail latency

Average latency hides slow users. p95/p99 show the slow tail. A system with 150 ms average but 3 s p99 may have a serious user problem that averages conceal.

## SLI, SLO, SLA

An SLI is a measured indicator, an SLO the reliability target, and an SLA a contractual promise. Example: `99.9% successful checkout requests monthly`. Error budgets turn the SLO into allowed unreliability.

## Safe deployment

Rolling deploy gradually replaces instances. Canary exposes a small percentage first. Blue/green keeps two environments and switches traffic. Mixed versions require backward-compatible API/database changes and reliable rollback.

## Authentication and authorization

Authentication answers who the caller is; authorization answers what they may do. Knowing a file ID must not imply permission to read the file. Services that own business state should enforce resource-level authorization.

## Encryption and secrets

TLS protects data in transit; encryption at rest protects stored data. Secrets, signing keys, and provider credentials belong in managed secret storage, never source code or logs.

## Privacy and lifecycle

PII needs retention, deletion, auditing, and access rules. Deleting one primary row is not enough if backups, indexes, analytics, and object copies still contain data.

## Abuse and isolation

Public systems face spam, scraping, abusive tenants, and credential attacks. Rate limits, quotas, validation, moderation, and tenant isolation reduce security and availability blast radius.

## Tokens, OAuth/OIDC, RBAC, and ACLs — recognition depth

Security questions in system design are usually about **trust boundaries and authorization**, not cryptographic implementation.

### Session cookies vs bearer tokens

A server-side session stores authentication state on the server and gives the client an opaque session identifier. A bearer token carries signed claims that services can validate without a central session lookup, though revocation becomes less immediate.

JWT is one common token format. Do not treat “JWT” as an authentication architecture by itself; you still need issuance, expiration, key rotation, audience/scope validation, and revocation strategy for sensitive cases.

### OAuth 2.0 and OpenID Connect

At recognition depth:

- **OAuth 2.0** is an authorization framework for delegated access;
- **OpenID Connect (OIDC)** adds an identity/authentication layer on top.

In interviews, saying “use OAuth” is not enough. State who the identity provider is, what token the client receives, and where authorization is enforced.

### RBAC and ACLs

**RBAC** grants permissions through roles such as admin/editor/viewer. **ACLs** attach permissions to a resource or object, such as a file shared with specific users.

Large systems often combine them: organizational roles for broad permissions plus resource-level ACLs for individual objects.

> **Important:** Verify identity at the edge if useful, but enforce resource authorization close to the service that owns the resource and understands its rules.

## Worked example — production checkout

Define a checkout success/latency SLO. Monitor traffic, errors, p95/p99, dependency latency, queue lag, and reconciliation mismatches. Canary releases; authenticate at edge, authorize order access in the owning service, use TLS/encryption, protect secrets, and audit sensitive state transitions.

## Interview lens

Mention the highest-risk operational/security concerns relevant to the prompt; do not dump every production topic into every interview.

## What to remember

A production-ready system is measurable, safely deployable, secure, recoverable, and governed—not only scalable.

## Review after reading

1. Metrics vs logs vs traces?
2. Why p99?
3. What is SLO?
4. What is canary?
5. Authentication vs authorization?

## Deeper study notes

### Observability should answer user questions

Start from “Can users complete checkout?” rather than infrastructure dashboards. Then map that to success rate, latency, payment error rate, inventory conflicts, and dependency health. CPU is useful only when it explains user impact or capacity.

### Alerts need actionability

An alert should indicate something requires human or automated action. Alerting on every single error creates noise. SLO burn rate, sustained error rate, queue lag, or capacity thresholds are usually more actionable than isolated events.

### Schema rollout uses expand-and-contract

To rename a database column safely: first add the new field and make code compatible with both; backfill; switch writers/readers; only later remove the old field. This avoids requiring every application instance to update atomically.

### Authorization follows the object

Edge auth can verify identity, but resource-level permission belongs where object ownership is known. For example, a gateway may verify JWT validity while the Drive service checks whether the user has `reader` permission on `file_id`.

### Privacy is a data-flow property

Map where sensitive data travels: primary DB, cache, logs, analytics, search, backups, object storage. Redaction and deletion requirements must cover those copies. Avoid logging secrets/tokens/PII by default.

### Common mistakes

- Monitoring only infrastructure and not business success.
- Using average latency instead of tail percentiles.
- Deploying incompatible DB changes before all old binaries are gone.
- Confusing authentication with authorization.
- Encrypting data but leaking it in logs or overly broad access controls.


## Important interview ideas

> **Important:** Production readiness should answer four questions: **Can we detect problems? Can we change the system safely? Can we prevent unauthorized use? Can we recover/comply when data or infrastructure fails?**

### Metrics: measure symptoms and causes

User-facing symptoms:

- success rate;
- p95/p99 latency;
- freshness/delivery delay;
- business success (checkout completion, message delivery).

System causes:

- CPU/memory;
- DB latency/connections;
- queue depth/age;
- cache hit rate;
- consumer lag;
- provider error rate.

A dashboard needs both. CPU at 40% does not prove users are healthy.

### Logs should be structured and safe

Include request/event ID, user/tenant ID when safe, operation, status, duration, dependency, and error class. Avoid secrets, full auth tokens, card data, and unnecessary PII.

Structured logs are searchable and can correlate with trace IDs.

### Traces reveal critical paths

A trace might show:

```text
checkout 800ms
 ├ inventory 80ms
 ├ payment 650ms  ← bottleneck
 └ DB 40ms
```

This is especially useful in multi-service systems where one request crosses several boundaries.

### SLO and error budget

If SLO is 99.9% availability, roughly 0.1% failures are allowed over the window. Error budget gives teams a reliability budget for change/risk.

Do not need exact SRE math in an L4 interview; show that alerts should connect to user-impacting objectives.

### Safe schema deployment

A classic expand/contract migration:

1. add new nullable column/table;
2. deploy code that can read old and new;
3. backfill;
4. switch writers/readers;
5. verify;
6. remove old field in a later release.

This avoids requiring every app instance to upgrade atomically.

### Authentication vs authorization

Authentication: who is the caller?
Authorization: may this caller perform this action on this object?

A gateway can validate a JWT, but Drive Service must check that the authenticated user has permission for `file_id`.

### Security boundaries

Consider:

- TLS in transit;
- encryption at rest where needed;
- secrets/key rotation;
- least-privilege service identities;
- input validation;
- rate limits/abuse;
- audit logs for sensitive actions.

### Privacy is data-flow wide

Deleting PII may involve:

```text
primary DB
cache
search index
analytics warehouse
object storage
backups/logs
```

Define lifecycle/retention rather than assuming primary-row deletion is enough.

## Worked scenario — checkout production review

After designing checkout, add:

**SLO:** 99.9% successful order submissions; p99 under agreed latency.

**Metrics:** order success, payment timeout, inventory conflict, queue backlog, reconciliation mismatch.

**Trace:** request across checkout→inventory→payment.

**Deployment:** canary new checkout version; backward-compatible schema.

**Security:** authenticate user, authorize cart/order ownership, protect payment tokens, audit status changes.

**Privacy:** minimize payment/PII retention and redact logs.

This is enough to show production thinking without turning the interview into an SRE/security audit.

## Interview questions and model answers

### Q1. “Metrics vs logs vs traces?”

Metrics provide aggregate trends/alerts, logs capture detailed discrete events, traces connect one request across services. I use metrics to know something is wrong, traces to locate the slow hop, and logs to inspect detailed context.

### Q2. “Why p99 instead of average?”

Average can hide a painful slow tail. User-facing systems often care that almost all requests meet the latency objective, so p95/p99 is more representative of degraded cohorts.

### Q3. “What is an SLO?”

A target for a measured service indicator, such as 99.9% successful requests or p99 latency below a threshold. It guides alerts, capacity, and reliability trade-offs.

### Q4. “Where should authorization happen?”

At the service that understands the resource and business rule. Edge token verification establishes identity, but object-level authorization should be enforced by the owner of that object/state.

## Common mistakes to avoid

- Only infrastructure metrics.
- Average latency only.
- Logging secrets/PII.
- Alert on every exception rather than user impact.
- Breaking DB schema deployments.
- Authentication confused with authorization.
- Encryption mentioned but access controls ignored.
- Deletion policy only for primary DB.

## Short revision note

Production checklist: **SLI/SLO → metrics/logs/traces → alert on user impact → safe rollout/rollback → authn/authz → encryption/secrets → privacy/retention → abuse/isolation**.

## Topics to revise

- [ ] metrics/logs/traces
- [ ] p95/p99
- [ ] SLI/SLO/SLA
- [ ] error budget
- [ ] canary/rolling/blue-green
- [ ] expand/contract migration
- [ ] authentication vs authorization
- [ ] privacy/retention/audit

## Interview-ready synthesis

### A strong 60–90 second explanation

I add production-readiness concerns selectively: user-facing SLOs and tail latency, metrics/logs/traces, safe canary/rollback and schema migration, authentication plus resource authorization, encryption/secrets, privacy lifecycle, and abuse/tenant isolation. I connect each concern to a real risk in the design.

### How this topic connects to the wider system

- Observability: symptoms + causes make failures diagnosable.
- Reliability: SLOs and safe rollout prevent change from silently harming users.
- Security: least privilege and object authorization protect data beyond authentication.
- Privacy/operations: retention/deletion must cover derived copies, logs, and backups.

### Revision flashcards with answers

**Metric?**  
Aggregated numeric signal such as QPS/error rate/p99/queue age.

**Trace?**  
One request's path/spans across services, useful for locating latency/failure.

**SLO?**  
Target for a measured service indicator, e.g. success rate/latency.

**Canary?**  
Send small traffic portion to new version before full rollout.

**Authn vs authz?**  
Authn establishes identity; authz decides permitted action/resource.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Checkout: SLO and trace make payment latency visible; audit protects financial state changes.
- Chat: message delivery lag/connection count are better business/service signals than CPU alone.
- File sharing: object authorization and signed URLs protect data even when CDN/object storage serves bytes.

### When not to overuse this idea

Do not turn the final system-design minutes into a generic security/SRE checklist. Mention the few production risks that materially follow from the prompt.

### A good interviewer sentence

> “I would use this only because the current requirement/workload creates the specific problem it solves. If that assumption changes, I would simplify or choose the alternative.”

This sentence captures an important L4 behavior: architecture is conditional, not dogmatic.

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

**Next:** Lesson 27 — Design a URL Shortener
