# Session 11 — SQL, Relational Databases & Transactions

## 1. Must Learn

### Relational fundamentals & SQL fit
- **Understand:** Understand tables, rows, schema, keys, relationships, joins, and constraints; know why SQL fits structured relational data and invariants.
- **Decision/trade-off:** Relational guarantees/query flexibility vs schema/distributed-scaling complexity.

### Normalization vs denormalization
- **Understand:** Know reducing duplication vs intentionally duplicating data for faster reads.
- **Decision/trade-off:** Consistency/write simplicity vs read efficiency.

### Transactions & boundaries
- **Understand:** Understand commit/rollback and grouping changes that must succeed or fail together; choose the smallest transaction protecting an invariant.
- **Decision/trade-off:** Atomic correctness vs contention/latency.

### ACID
- **Understand:** Understand Atomicity, Consistency, Isolation, Durability; ACID consistency means preserving invariants, not distributed consistency.
- **Decision/trade-off:** Guarantee strength vs cost/concurrency.

### Isolation & anomalies
- **Understand:** Understand dirty read, non-repeatable read, phantom read; know Read Committed, Repeatable Read, Serializable conceptually.
- **Decision/trade-off:** Stronger isolation/correctness vs concurrency/blocking/retries.

### Contention, deadlocks & retries
- **Understand:** Know long/conflicting transactions reduce throughput and deadlocks may cause abort/retry.
- **Decision/trade-off:** Larger/stronger transactions vs concurrency and availability.

## 2. Should Know

- Application validation vs database constraints.
- Many-to-many relationships/junction tables.
- Avoid slow network work inside database transactions when possible.

## 3. Recognition Only

- MVCC
- Snapshot isolation
- Optimistic vs pessimistic concurrency control
- Row locks

## 4. Important Comparisons

- SQL vs non-relational storage — focus on relationships, transactions, constraints, access patterns.
- Normalization vs denormalization.
- Application checks vs database constraints.
- Read Committed vs Repeatable Read vs Serializable.
- Short vs long transactions.

## 5. Important Interview Questions

1. Why choose SQL for this system?
2. What invariant does this transaction protect?
3. Where should the transaction boundary be?
4. What does ACID mean?
5. What can go wrong with two concurrent writers?
6. When is Read Committed enough, and when is stronger isolation needed?
7. Why not use Serializable everywhere?
8. What happens on deadlock/conflict?

## 6. Common Interview Mistakes

- **“SQL doesn't scale”** → Choose storage by requirements first; scaling is a separate problem.
- **“Use Serializable for safety”** → Use the minimum isolation that protects correctness.
- **Huge transaction around an entire request** → Keep the transaction small and short.
- **Ignoring concurrent writers** → Reason explicitly about races.
- **Only application-level checks for critical invariants** → Use database constraints/transactions where needed.
- **Confusing ACID consistency with distributed consistency** → Treat them as different concepts.

## 7. Communication

### Important Vocabulary

transaction, invariant, constraint, primary key, foreign key, normalization, denormalization, ACID, isolation level, contention, deadlock, rollback, serializable

### Useful Interview Phrases

- “The invariant I need to protect is …”
- “I’d keep the transaction boundary as small as possible.”
- “I only need stronger isolation if concurrent operations can violate this invariant.”
- “The trade-off is stronger correctness at the cost of concurrency.”

### Important Questions to Ask the Interviewer

- **Question:** “Can multiple requests modify this resource concurrently?”  
  **Why it matters:** Determines isolation/concurrency handling.
- **Question:** “Which updates must succeed or fail together?”  
  **Why it matters:** Determines transaction boundary.
- **Question:** “How strict does correctness need to be here?”  
  **Why it matters:** Determines guarantee strength.

## 8. ⭐ Must Remember

1. Transactions protect invariants, not just groups of queries.
2. Keep transactions small and short.
3. ACID consistency ≠ distributed consistency.
4. Concurrent writes can break apparently correct application logic.
5. Isolation is a correctness-vs-concurrency trade-off.
6. Do not automatically choose Serializable.
7. Critical invariants often need database-level protection.
8. Design for aborts/deadlocks and retries.

## 9. Study Priority

1. Study first: SQL fit, relational model, constraints, normalization.
2. Study next: transactions, boundaries, ACID.
3. Finish with: anomalies, isolation levels, contention, deadlocks.

## 10. Revision Checklist

- [ ] Explain when SQL is appropriate.
- [ ] Identify an invariant and transaction boundary.
- [ ] Explain ACID and common anomalies.
- [ ] Compare main isolation levels.
- [ ] Discuss normalization, contention, deadlocks, and retries.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
