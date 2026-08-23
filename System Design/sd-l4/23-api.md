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
