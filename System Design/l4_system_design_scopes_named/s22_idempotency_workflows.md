# Session 22 — Idempotency, Concurrency & Distributed Workflows

## 1. Must Learn

### Race conditions & concurrency control
- **Understand:** Understand concurrent updates can violate invariants; know optimistic locking/version checks, pessimistic locking, compare-and-set, and unique constraints conceptually.
- **Decision/trade-off:** Higher concurrency/retries vs blocking/lock cost.

### Idempotency
- **Understand:** Use idempotency keys or idempotent operation design so retries/duplicates do not repeat logical side effects.
- **Decision/trade-off:** Correct retry behavior vs dedup state/retention complexity.

### Leases & distributed locks
- **Understand:** Understand time-bounded ownership for coordination, expiry, and clock/stale-holder risks.
- **Decision/trade-off:** Coordination simplicity vs liveness/correctness hazards.

### Fencing tokens
- **Understand:** Know a monotonically newer token can stop stale lease holders from committing after losing ownership.
- **Decision/trade-off:** Stronger stale-writer protection vs downstream enforcement complexity.

### Dual-write problem & transactional outbox
- **Understand:** Understand updating a database and publishing an event separately can leave inconsistent state; outbox couples intent with the DB transaction.
- **Decision/trade-off:** Consistency/recovery vs extra relay/storage machinery.

### Saga & compensation
- **Understand:** Coordinate multi-service workflows with local transactions and compensating actions instead of one global transaction.
- **Decision/trade-off:** Availability/service autonomy vs intermediate states/compensation complexity.

### Reconciliation
- **Understand:** Use periodic comparison/repair when distributed workflows can end in uncertain or divergent states.
- **Decision/trade-off:** Operational safety net vs delayed correction.

## 2. Should Know

- CDC conceptually as a way to capture committed changes/events.
- Leader election recognition only.
- CQRS/event sourcing recognition only.
- Unique constraints are often simpler than distributed locks for uniqueness.

## 3. Recognition Only

- Two-phase commit
- Consensus internals
- Event sourcing internals

## 4. Important Comparisons

- Optimistic vs pessimistic locking.
- Compare-and-set/version check vs distributed lock.
- Idempotency key vs deduplication by business identifier.
- Lease/lock vs fencing token.
- Direct dual write vs transactional outbox.
- Saga/compensation vs single-database transaction.

## 5. Important Interview Questions

1. How do we prevent charging/creating the same thing twice?
2. What if two clients update the same record concurrently?
3. What if a worker pauses until after its lease expires and then resumes?
4. Why may a distributed lock alone be insufficient?
5. What if the DB commit succeeds but event publish fails?
6. How does the workflow recover from partial failure?
7. What reconciles uncertain outcomes later?

## 6. Common Interview Mistakes

- **Locking before trying simpler constraints/CAS** → Use the simplest correctness primitive that protects the invariant.
- **Idempotency means “ignore duplicates forever”** → Define key scope, result reuse, and retention.
- **Distributed lock = guaranteed correctness** → Handle lease expiry/stale holders; fencing may be needed.
- **Naive DB + broker dual write** → Use outbox/CDC/reconciliation strategy.
- **Saga as rollback** → Compensation is a business action and may not perfectly undo reality.

## 7. Communication

### Important Vocabulary

race condition, optimistic locking, pessimistic locking, compare-and-set, version, idempotency key, lease, distributed lock, fencing token, saga, compensation, transactional outbox, CDC, reconciliation, dual write

### Useful Interview Phrases

- “I’ll protect the invariant with the simplest primitive that works—preferably a constraint or version check before a distributed lock.”
- “Retries are expected, so this side effect must be idempotent.”
- “A fencing token prevents an expired lease holder from committing stale work.”
- “The outbox avoids the DB-plus-broker dual-write gap.”

### Important Questions to Ask the Interviewer

- **Question:** “What invariant must never be violated?”  
  **Why it matters:** Determines concurrency primitive.
- **Question:** “Can requests/events be delivered more than once?”  
  **Why it matters:** Determines idempotency.
- **Question:** “Can this workflow span multiple data owners/services?”  
  **Why it matters:** May require saga/outbox/reconciliation.

## 8. ⭐ Must Remember

1. Start from the invariant.
2. Retries and duplicate delivery are normal.
3. Prefer simple constraints/CAS before distributed locks when sufficient.
4. Leases expire; stale holders are dangerous.
5. Fencing protects against stale writers.
6. DB + broker dual writes need a consistency strategy.
7. Sagas use compensation, not magic rollback.
8. Reconciliation is a practical recovery tool.

## 9. Study Priority

1. Study first: race conditions, constraints/CAS/optimistic locking, idempotency.
2. Study next: leases, stale holders, fencing.
3. Finish with: dual-write, outbox/CDC, saga, reconciliation.

## 10. Revision Checklist

- [ ] Protect an invariant under concurrent requests.
- [ ] Design an idempotent retry path.
- [ ] Explain lease-expiry stale-worker failure.
- [ ] Explain fencing tokens.
- [ ] Handle DB+event dual write.
- [ ] Explain saga/compensation and reconciliation.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
