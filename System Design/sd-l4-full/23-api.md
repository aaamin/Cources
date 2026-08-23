# Lesson 23 — API & Event Contract Design

**Phase:** Fundamentals  
**Session:** 23/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn REST resource design, methods, pagination, versioning, errors, idempotency, gRPC concepts, webhooks, and stable event contracts.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Resource-oriented APIs

Model domain nouns cleanly: `POST /v1/messages`, `GET /v1/messages/{id}`, `GET /v1/conversations/{id}/messages`. Avoid endpoints that expose internal service topology rather than the domain.

## HTTP methods

GET reads; POST creates/triggers; PUT usually replaces a known resource and is idempotent; PATCH partially updates; DELETE removes/marks removed. Exact conventions vary, but consistent semantics help clients reason about retries.

## Pagination

Offset pagination is simple but expensive/unstable at large offsets on changing datasets. Cursor pagination uses a stable ordering key such as `(created_at,id)` and is natural for feeds, messages, and timelines.

## Errors and idempotency

Contracts should distinguish invalid input, unauthenticated/unauthorized, conflict, rate limit, and transient server failure. A client must know whether retry is safe. Idempotency keys make retry semantics explicit for create/payment-style requests.

## Versioning and compatibility

APIs evolve while old clients remain. Path/header versioning is less important than maintaining compatible behavior during mixed-version rollout. Schema changes should be additive before destructive cleanup.

## gRPC, webhooks, events

gRPC offers typed RPC/streaming and is common internally. Webhooks call external systems when events happen and require signatures, retries, dedupe, and replay protection. Events should have stable IDs, type/version, time, and clear schema ownership.

## Worked example — message API

Use `POST /v1/messages` with an idempotency key and `GET /v1/conversations/{id}/messages?cursor=...`. Return a `next_cursor`, not only page numbers. Define what duplicate POST retries return and how auth is checked.

## Interview lens

You usually need only 3–5 important contracts in an interview. Pick endpoints that expose the core reads and state transitions.

## What to remember

Good contracts make identity, pagination, errors, retry, compatibility, and event semantics explicit.

## Review after reading

1. Why cursor pagination?
2. PUT vs POST conceptually?
3. Why classify errors?
4. Why compatibility/versioning?
5. What security issue with webhooks?

## Deeper study notes

### Model errors as part of the API

A successful shape is not enough. Clients need stable meanings for `400` validation, `401` unauthenticated, `403` unauthorized, `404` missing, `409` conflict, `429` throttled, and `5xx` transient server failure. Exact status usage varies, but retryability should be unambiguous.

### Cursor design must be opaque enough to evolve

A cursor may encode `(created_at,id)` or an internal token. Clients should normally treat it as opaque. This lets the server change pagination internals without breaking API contracts.

### Idempotent creation needs request identity

For an API such as `POST /payments`, the server cannot infer whether two identical payloads are two intended payments or one retry. A caller-provided idempotency key supplies logical request identity.

### Webhooks are distributed-system calls to someone else's infrastructure

Sign requests, include unique event IDs, retry with backoff, and expect consumer downtime. The receiver should respond quickly and process asynchronously. Provide replay or delivery logs when the product requires reliable integration.

### Common mistakes

- Designing dozens of endpoints before the architecture.
- Returning offset pagination for a high-churn billion-row feed without considering instability.
- Changing response fields incompatibly while old clients remain.
- Treating authentication token validity as resource authorization.


## Important interview ideas

> **Important:** An API contract is part of correctness. It must make identity, pagination, retries, conflicts, and asynchronous behavior understandable to clients.

### Resource modeling

Prefer resource nouns and stable identifiers:

```http
POST /v1/orders
GET  /v1/orders/{order_id}
PATCH /v1/orders/{order_id}
```

Avoid endpoint names that expose internal implementation such as `/callOrderDatabase`.

### Idempotent create APIs

POST is not inherently idempotent, but you can make a logical create safely retryable with an idempotency key.

Request:

```http
POST /v1/orders
Idempotency-Key: 6fa...
```

The server stores the result associated with that key. Retries return the same order rather than creating another.

### Cursor pagination in depth

Offset pagination:

```text
OFFSET 1,000,000 LIMIT 50
```

can be expensive and unstable under inserts/deletes.

Cursor pagination encodes the last ordered item:

```text
(created_at=..., id=...)
```

Next query asks for rows after/before that stable boundary. Include a tie-breaker ID because timestamps can collide.

### Error semantics

Clients need to distinguish:

- 400-style invalid input → do not retry unchanged;
- 401/403 → authentication/authorization issue;
- 404 → resource absent;
- 409 → conflict/state race;
- 429 → rate limited, retry later;
- 5xx/timeout → potentially transient, retry according to idempotency policy.

Do not obsess over exact codes in system design; explain retry semantics.

### Webhooks

A webhook is reverse delivery: your system sends an HTTP request to a customer's endpoint. Treat it like an at-least-once notification:

- sign payload;
- include event ID/timestamp;
- retry with backoff;
- protect against SSRF when accepting callback URLs;
- consumer deduplicates/rejects replay.

### Event contracts

Events should include:

```text
event_id
event_type
schema_version
occurred_at
aggregate/entity id
payload/reference
```

Schema changes should be backward compatible because consumers deploy independently.

## Interview questions and model answers

### Q1. “Offset or cursor pagination?”

Offset is simple for small/stable datasets. Cursor is better for large, frequently changing ordered lists because it avoids deep skip cost and reduces duplicates/missing rows caused by inserts between pages.

### Q2. “How do clients retry POST safely?”

Use an idempotency key tied to the logical operation. Store request identity/result for a retention window and reject key reuse with conflicting input.

### Q3. “REST or gRPC?”

REST/HTTP+JSON is simple and broadly interoperable for public APIs. gRPC is strong for typed internal RPC, code generation, and streaming. I choose based on clients, performance, schema tooling, and operational ecosystem rather than ideology.

### Q4. “How do webhooks handle failure?”

Persist the event, send with a stable ID/signature, retry transient failures with backoff, expose delivery status, and eventually DLQ/stop. Receivers must dedupe because timeout can occur after they processed the event.

## Common mistakes to avoid

- Too many endpoints before architecture.
- No pagination for unbounded list.
- Offset pagination assumed scalable forever.
- POST retries with no idempotency.
- Errors with no retry semantics.
- Webhooks with no signature/dedupe.
- Breaking event schemas silently.

## Short revision note

API checklist: **resource + identity + pagination + error/retry + idempotency + version compatibility**. Event/webhook checklist adds **event ID + signature/version + at-least-once behavior**.

## Topics to revise

- [ ] REST resources/methods
- [ ] cursor pagination
- [ ] idempotency key
- [ ] conflict/rate-limit errors
- [ ] versioning/backward compatibility
- [ ] gRPC concept
- [ ] webhooks
- [ ] event contracts

## Interview-ready synthesis

### A strong 60–90 second explanation

I keep API contracts small and explicit. Resource identity, pagination order, retry/error behavior, idempotency, and backward compatibility are part of design correctness. For webhooks/events I add stable event IDs, signatures/versioning, and at-least-once delivery assumptions.

### How this topic connects to the wider system

- Correctness: idempotency/error semantics keep client retries safe.
- Performance: cursor pagination scales better for deep changing lists.
- Security: webhook signatures and object authorization prevent spoofing/leakage.
- Evolution: version-compatible contracts allow independent client/service deployment.

### Revision flashcards with answers

**Cursor?**  
Opaque/stable position based on last ordered item rather than numeric page offset.

**Why tie-breaker ID?**  
Multiple rows can share timestamp, so ID gives deterministic order.

**429?**  
Rate limited; client should back off according to policy/Retry-After.

**Webhook retry concern?**  
Receiver may process then response is lost, so event IDs/dedupe are required.

**Versioning goal?**  
Allow old and new clients/consumers to coexist during evolution.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Feed/history: cursor pagination follows stable sort key rather than offset.
- Payment: POST with idempotency key makes timeout retry safe.
- Webhook: stable event ID + signature + retry gives receiver an auditable at-least-once contract.

### When not to overuse this idea

Do not spend interview time designing dozens of endpoints. A few contracts exposing main state transitions and access patterns are enough.

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

**Next:** Lesson 24 — Real-Time Communication
