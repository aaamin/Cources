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
