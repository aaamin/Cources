# Session 23 — API, Authentication & Event Contract Design

## 1. Must Learn

### REST resource design
- **Understand:** Model resources and use HTTP methods with clear semantics.
- **Decision/trade-off:** Simple predictable API vs action-oriented endpoints when resource model is awkward.

### Pagination/filtering/sorting
- **Understand:** Design bounded list endpoints and know cursor vs offset pagination.
- **Decision/trade-off:** Simplicity/random access vs stability/performance at changing/high-scale data.

### Errors & retry behavior
- **Understand:** Make failure status, retryability, and duplicate behavior explicit.
- **Decision/trade-off:** Useful client recovery vs leaking internals/ambiguous outcomes.

### Idempotency keys
- **Understand:** Use for retryable side-effecting operations when duplicate execution is dangerous.
- **Decision/trade-off:** Safe retries vs server-side dedup/result retention.

### API versioning & compatibility
- **Understand:** Evolve contracts without breaking existing clients.
- **Decision/trade-off:** Clean breaking changes vs compatibility burden.

### Event contracts & schema evolution
- **Understand:** Events are long-lived contracts consumed independently; evolve additively/compatibly when possible.
- **Decision/trade-off:** Producer evolution vs consumer breakage.

### Authentication mechanisms at selection depth
- **Understand:** Recognize cookie/session, bearer token/JWT, API key, OAuth 2.0/OIDC, and service-to-service auth and their appropriate roles.
- **Decision/trade-off:** Convenience/scalability/delegation vs revocation/security complexity.

## 2. Should Know

- gRPC conceptually for typed service-to-service RPC.
- Webhooks as outbound event notifications requiring retry/idempotency/security.
- Rate-limit responses should communicate throttling clearly.
- Authorization is distinct from authentication even though deeper security is Session 26.

## 3. Recognition Only

- GraphQL
- mTLS details
- OAuth grant-flow internals

## 4. Important Comparisons

- POST vs PUT/PATCH.
- Offset vs cursor pagination.
- REST vs gRPC conceptually.
- Webhook vs polling.
- Cookie/session vs bearer token.
- API key vs OAuth/OIDC user-delegated identity.
- Request schema evolution vs event schema evolution.

## 5. Important Interview Questions

1. What are the core resources/actions?
2. Can this write be retried safely?
3. How large/fast-changing is the collection being paginated?
4. How should clients distinguish retryable from permanent errors?
5. Who is authenticating: end user, client app, or service?
6. How will old consumers handle a new event field/version?

## 6. Common Interview Mistakes

- **Unbounded list endpoint** → Paginate.
- **Offset pagination for rapidly changing huge feeds automatically** → Consider cursor-based pagination.
- **Ambiguous retries on POST** → Use idempotency where duplicate side effects matter.
- **JWT for every auth problem** → Choose mechanism based on identity/delegation/revocation needs.
- **Breaking event schemas casually** → Assume independent consumers upgrade at different times.

## 7. Communication

### Important Vocabulary

REST, resource, GET, POST, PUT, PATCH, DELETE, cursor, offset, pagination, idempotency key, API versioning, webhook, event contract, bearer token, JWT, API key, OAuth 2.0, OpenID Connect

### Useful Interview Phrases

- “I’ll make retry and duplicate behavior explicit in the API contract.”
- “For a changing large dataset, cursor pagination gives more stable traversal.”
- “The event schema needs backward-compatible evolution because consumers deploy independently.”

### Important Questions to Ask the Interviewer

- **Question:** “Is this API public/external or internal?”  
  **Why it matters:** Changes compatibility/auth expectations.
- **Question:** “Can clients retry writes after timeout?”  
  **Why it matters:** Determines idempotency design.
- **Question:** “Are list results large and frequently changing?”  
  **Why it matters:** Determines pagination strategy.

## 8. ⭐ Must Remember

1. APIs should make errors, retries, duplicates, pagination, and auth explicit.
2. Use bounded pagination.
3. Cursor pagination is often better for large changing ordered datasets.
4. Idempotency keys make dangerous retries safer.
5. Event contracts must evolve compatibly.
6. Authentication mechanism depends on who/what is authenticating.

## 9. Study Priority

1. Study first: REST resources/methods, errors, pagination.
2. Study next: idempotency, versioning, event contracts.
3. Finish with: auth mechanism recognition, gRPC, webhooks.

## 10. Revision Checklist

- [ ] Design clean endpoints for a small domain.
- [ ] Choose cursor vs offset pagination.
- [ ] Define retry/idempotency behavior.
- [ ] Explain API/event compatibility.
- [ ] Choose an appropriate auth mechanism conceptually.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
