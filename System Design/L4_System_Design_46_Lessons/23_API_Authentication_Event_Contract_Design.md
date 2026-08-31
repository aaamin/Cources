# Session 23 — API, Authentication & Event Contract Design

## Outcome

You should be able to design clear REST-like APIs, pagination, filtering/sorting, errors, versioning, idempotency behavior, basic gRPC/webhook/event contracts, and recognize session, bearer/JWT, API-key, OAuth, OIDC, and service-to-service authentication patterns.

## API Design Starts From Resources and Actions

For a storage service:

```http
POST   /files
GET    /files/{id}
DELETE /files/{id}
GET    /folders/{id}/files?cursor=...
```

Avoid RPC-like endpoints everywhere:

```http
POST /doCreateFile
POST /getFile
```

But REST purity is not the goal. Clear semantics are.

Some domain operations are naturally actions:
```http
POST /orders/{id}/cancel
```
can be perfectly reasonable.

## HTTP Methods

### GET
Read. Should be safe and normally idempotent.

### POST
Create/trigger action. Not inherently idempotent.

### PUT
Replace/create at known resource URI; conceptually idempotent.

### PATCH
Partial update; idempotency depends on semantics.

### DELETE
Delete; conceptually idempotent if repeated delete leaves same final state.

Do not rely only on method name for business idempotency.

## Resource Naming

Prefer nouns and hierarchy where it helps.

```text
/users/{id}/orders
/conversations/{id}/messages
```

Avoid extreme nesting:
```text
/users/1/orgs/2/projects/3/tasks/4/comments/5
```

Resource IDs can be passed directly once ownership/authorization is checked.

## Pagination

Never return unbounded large collections.

### Offset pagination

```http
?page=1000&size=50
```

Simple.

Problems:
- large offsets may be expensive;
- inserts/deletes between pages cause duplicates/skips.

### Cursor pagination

```http
?cursor=eyJ0aW1lIj...
```

Cursor encodes last ordering position.

Good for:
- feeds;
- messages;
- large ordered datasets.

Requires stable deterministic sort, e.g.:

```text
(created_at, id)
```

Cursor should be opaque to client where possible.

## Filtering and Sorting

Example:

```http
GET /orders?status=pending&sort=-created_at&cursor=...
```

Only expose supported filters that can be served efficiently/indexed.

Do not allow arbitrary client-generated SQL-like filters unless system is designed for it.

## Error Design

Use consistent error shape:

```json
{
  "code": "ORDER_ALREADY_CANCELLED",
  "message": "..."
}
```

Important distinctions:
- validation;
- authentication;
- authorization;
- not found;
- conflict;
- rate limit;
- dependency unavailable.

Clients need to know whether retry is safe.

## Idempotency Contract

For money/order creation:

```http
POST /payments
Idempotency-Key: ...
```

Document:
- key scope;
- retention;
- same key/different payload behavior;
- in-progress request behavior;
- returned status.

This is part of API design, not an internal implementation detail.

## API Versioning

Strategies:
- path `/v2/...`;
- header/content negotiation;
- backward-compatible evolution without frequent versions.

Prefer additive compatible changes:
- add optional field;
- add endpoint;
- preserve old semantics.

Breaking changes need migration period.

## gRPC Recognition

gRPC-style RPC can be useful for service-to-service calls:
- typed contracts;
- efficient binary serialization;
- streaming support.

Trade-offs:
- browser/public API ergonomics differ;
- schema/versioning still matters;
- network failures remain.

Do not say gRPC makes distributed calls reliable automatically.

## Webhooks

Server calls customer's endpoint when event occurs.

Flow:
```text
Your system → Customer webhook URL
```

Need:
- authentication/signature;
- event ID;
- retry;
- backoff;
- timeout;
- DLQ/failed delivery state;
- duplicate delivery;
- ordering expectations;
- replay/manual retry;
- endpoint security.

Customer must treat webhook idempotently.

## Event Contract

Example:

```json
{
  "eventId": "e123",
  "type": "OrderPaid",
  "version": 2,
  "occurredAt": "...",
  "orderId": "o1",
  "paymentId": "p1"
}
```

Good events include:
- unique event ID;
- event type/version;
- business identifiers;
- timestamp as informational ordering, not always perfect global order;
- enough data/reference for consumer.

Avoid event names like:
```text
UpdateData
```

## Event Schema Evolution

Events may be replayed months later.

Prefer:
- additive fields;
- default behavior for missing fields;
- version-aware consumers;
- schema registry if ecosystem warrants it.

Do not remove/rename fields immediately while old consumers exist.

## Authentication vs Authorization

Authentication:
> Who are you?

Authorization:
> Are you allowed to do this?

Do not conflate.

## Cookie/Session Authentication

Browser stores session cookie; server/session store maps session to user.

Pros:
- server can revoke session centrally;
- familiar web pattern.

Costs:
- server-side session state/storage;
- CSRF considerations for cookie-authenticated actions.

## Bearer Token / JWT Recognition

Client sends:
```http
Authorization: Bearer <token>
```

JWT may contain signed claims and be validated without central lookup.

Benefits:
- stateless validation;
- useful across services.

Costs:
- revocation is harder until expiry unless extra mechanism;
- token size;
- leaked bearer token can be used by holder;
- claims can become stale.

Do not put secrets/sensitive data casually in JWT payload; encoding is not encryption.

## API Keys

Useful for:
- service/customer application identification;
- machine clients;
- developer APIs.

Need:
- secure storage;
- rotation;
- scoping;
- rate limits;
- not expose in URLs/logs.

## OAuth 2.0 Concept

Authorization framework for delegated access.

Example:
> Allow app X to access your Google profile without giving app X your Google password.

At L4 system-design depth, understand the role:
- authorization server;
- client;
- user/resource owner;
- access token;
- scopes.

You do not need to memorize every grant unless target role requires it.

## OpenID Connect Concept

Identity layer on top of OAuth-style authorization.

Used for login/SSO identity information.

Mental distinction:
```text
OAuth → delegated authorization
OIDC → authentication/identity on top
```

## Service-to-Service Authentication

Internal traffic still needs trust.

Options conceptually:
- mTLS;
- service identity tokens;
- workload identity;
- signed credentials.

Use least privilege:
- service A should only access APIs/resources it needs.

## Worked Example — Create Order API

```http
POST /orders
Authorization: Bearer ...
Idempotency-Key: abc
```

Request:
```json
{
  "items":[...],
  "shippingAddressId":"..."
}
```

Server:
- authenticate user;
- authorize address/items;
- atomically claim idempotency key;
- create order;
- return `201`.

Duplicate same key:
- same request → same logical result;
- different payload → conflict/error.

List:
```http
GET /orders?cursor=...
```

Uses stable cursor.

Order events:
```text
OrderCreated v1
OrderPaid v1
```

## Small Design Drills

1. Why is cursor pagination better for a fast-changing feed?
2. Does JWT automatically solve authorization?
3. Why must webhooks be idempotent?
4. OAuth vs OIDC in one sentence?
5. Why shouldn't an API key go in a URL query string?
6. What should happen if same idempotency key is reused with different payload?
7. Why is a timestamp alone often a weak cursor?

<details>
<summary>Answer key</summary>

1. Stable seek-based position avoids huge offsets and reduces duplicates/skips under inserts.
2. No. JWT can authenticate/transport claims; server still enforces authorization.
3. Delivery retries/duplicates are normal.
4. OAuth is delegated authorization; OIDC adds identity/authentication semantics.
5. URLs leak into logs/history/referrers more easily.
6. Reject as conflict/invalid reuse rather than silently performing different operation.
7. Multiple items can share timestamp; use deterministic tie-breaker such as ID.

</details>

## Common Interview Mistakes

- Spending 10 minutes on endpoint syntax.
- Unbounded list API.
- Offset pagination at enormous depth.
- JWT = encryption.
- Authentication = authorization.
- Webhook without signature/retry/idempotency.
- Breaking event schemas with no compatibility.
- gRPC = reliable.
- Idempotency key with undefined scope/retention.
- OAuth described simply as “login.”

## Must Remember

- **API semantics matter more than REST purity.**
- **Large collections need pagination.**
- **Cursor pagination needs stable deterministic order.**
- **Errors should tell clients what happened/retryability.**
- **Idempotency is part of contract for retry-sensitive writes.**
- **Webhooks duplicate and need signatures/retries.**
- **Event contracts must survive replay/schema evolution.**
- **Authentication identifies; authorization permits.**
- **JWT is signed claims, not automatically encrypted/revocable.**
- **OAuth is authorization; OIDC adds identity.**

## Interview Revision Summary

API checklist:

```text
Core resources/actions?
Methods?
Auth?
Authorization?
Pagination?
Filter/sort?
Errors?
Retry?
Idempotency?
Versioning?
Rate limits?
Webhook?
Event ID/version/schema?
```

## Explain Without Notes

Design APIs/events for an order service including create-order idempotency, paginated history, authentication, and an `OrderPaid` webhook/event.

## Completion Checklist

- [ ] I can design concise APIs.
- [ ] I understand cursor vs offset pagination.
- [ ] I define errors/idempotency/versioning.
- [ ] I understand webhook semantics.
- [ ] I distinguish authn/authz.
- [ ] I recognize session/JWT/API key/OAuth/OIDC patterns.
