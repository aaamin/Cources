# Session 11 — SQL, Relational Databases & Transactions

## Outcome

You should understand relational modeling, keys/constraints/joins, normalization and denormalization, ACID transactions, isolation anomalies, common isolation levels, deadlocks, and when relational databases are the safest default.

## Why Relational Databases Are Often a Good Default

Relational databases are strong when your domain has:

- structured entities;
- relationships;
- uniqueness constraints;
- multi-row invariants;
- transactional updates;
- flexible queries/joins.

Examples:
- orders/payments;
- inventory;
- subscriptions;
- employee/account systems.

Choosing SQL does not mean the system cannot scale. Replication, partitioning, caching, and managed clusters can take relational systems very far.

## Tables, Rows and Relationships

Example:

```text
User(id, name, email)
Order(id, user_id, status, total)
OrderItem(order_id, product_id, qty, price)
```

Foreign keys express relationships.

Primary key:
- uniquely identifies a row.

Unique constraint:
- enforces uniqueness beyond the primary key.

Example:
```text
UNIQUE(email)
```

This is stronger for correctness than “check then insert” in application code under concurrency.

## Normalization

Normalization reduces duplication and anomalies.

Example instead of storing customer email on every order:
```text
Order.user_id → User.id
```

Benefits:
- one source for user information;
- easier updates;
- fewer inconsistencies.

Costs:
- reads may require joins;
- distributed/sharded joins can become difficult.

## Denormalization

Duplicate/precompute data for faster reads.

Example:
```text
Order.customer_name_at_purchase
```

This may be intentionally historical and avoids a join.

Or feed system:
- precompute user feed entries.

Trade:
- faster read;
- extra storage;
- update/reconciliation complexity.

## Joins

Relational databases excel at joining related data.

But expensive joins over huge unindexed datasets or across shards can be problematic.

Do not say “joins are bad.” Ask:
- data size?
- indexes?
- selectivity?
- query frequency?
- shard locality?

## Constraints as Correctness Tools

Useful constraints:
- primary key;
- unique key;
- foreign key;
- check constraint;
- not null.

For race-prone uniqueness, let the database enforce the invariant.

Example: coupon redemption only once:
```text
UNIQUE(user_id, coupon_id)
```

Now two concurrent requests cannot both insert successful redemption rows.

## Transactions

A transaction groups operations into one unit.

Example bank transfer:

```text
BEGIN
debit A
credit B
record ledger
COMMIT
```

Either all changes commit or none should.

## ACID

### Atomicity
All operations in transaction commit together or roll back.

### Consistency
Transaction moves database from one valid state to another assuming rules/constraints are correct. Do not confuse this ACID “consistency” with distributed consistency models.

### Isolation
Concurrent transactions behave according to an isolation contract.

### Durability
Once committed, data survives failures according to database guarantees.

## Isolation Problems

Imagine transactions run concurrently.

### Dirty read
Transaction B reads uncommitted data from A.

If A rolls back, B observed something that never became real.

### Non-repeatable read
Transaction reads one row twice and sees different committed values because another transaction updated it.

### Phantom read
A repeated range query returns a different set of rows due to concurrent insert/delete.

These are conceptual interview-level anomalies.

## Isolation Levels

Exact DB behavior varies, but use this mental model.

### Read Committed
Only committed data is read. Common default.

Still allows values/results to change between statements.

### Repeatable Read
Repeated reads of already-seen rows are more stable within transaction. Exact phantom behavior depends on DB implementation.

### Serializable
Strongest common isolation: outcome behaves like transactions ran serially in some order.

Benefits:
- easiest reasoning for invariants.

Costs:
- more blocking/aborts/retries;
- lower concurrency for contentious workloads.

Do not choose serializable for every operation automatically.

## Lost Update

Two requests:

```text
A reads stock=10
B reads stock=10
A writes stock=9
B writes stock=9
```

Expected stock after two purchases = 8, but result = 9.

Solutions:
- atomic update: `stock = stock - 1 WHERE stock > 0`;
- row lock;
- optimistic version check;
- serializable transaction;
- database constraint/design.

This connects SQL transactions to the later concurrency lesson.

## Deadlocks

Transaction A locks row X then wants Y.
Transaction B locks Y then wants X.

```text
A: X → waits Y
B: Y → waits X
```

Database detects a deadlock and aborts one transaction.

Applications should:
- keep transactions short;
- lock resources in consistent order where possible;
- retry deadlock victims safely.

## Long Transactions

Long transactions:
- hold locks/version history;
- increase contention;
- increase deadlock chance;
- reduce throughput.

Do not place a slow external HTTP call inside a DB transaction if avoidable.

Bad:

```text
BEGIN
lock order
call payment provider for 10 sec
update
COMMIT
```

Distributed workflows require Saga/reconciliation rather than holding DB locks across external systems.

## Worked Example — Seat Reservation

Need:
- never reserve same seat twice.

Schema:
```text
Seat(event_id, seat_id, status, hold_id, hold_expires_at)
UNIQUE(event_id, seat_id)
```

Possible transaction:
```sql
UPDATE Seat
SET status='HELD', hold_id=:id
WHERE event_id=:event
  AND seat_id=:seat
  AND status='AVAILABLE';
```

Check affected row count.

This atomic conditional update avoids “read available, then later write held” race.

Payments happen outside a long DB transaction; later workflow handles success/failure/expiry.

## SQL vs “NoSQL Because Scale”

Do not discard relational DB because traffic is high.

Ask:
- do transactions/invariants matter?
- are access patterns relational?
- can reads use indexes/cache/replicas?
- can writes be partitioned later?
- is one cluster actually insufficient?

Correctness often matters more than fashionable scale claims.

## Small Design Drills

1. Why is `SELECT if email exists` then `INSERT` unsafe for uniqueness under concurrency?
2. What ACID property ensures committed data survives crash?
3. Why should external payment calls usually not happen while holding DB locks?
4. Two users buy last inventory unit concurrently. Name three DB-level approaches.
5. What does serializable trade for easier correctness reasoning?
6. Is denormalization always a NoSQL technique?

<details>
<summary>Answer key</summary>

1. Both transactions can see “not exists” before either inserts; a unique constraint is authoritative.
2. Durability.
3. Slow/unreliable network calls prolong transaction/locks and amplify contention/failure.
4. Conditional atomic update, row lock/pessimistic transaction, optimistic version, serializable isolation.
5. Concurrency/throughput; transactions may block or abort/retry.
6. No. Relational systems also denormalize/precompute strategically.

</details>

## Common Interview Mistakes

- Confusing ACID consistency with CAP consistency.
- Saying SQL cannot horizontally scale.
- Saying joins are inherently slow.
- Enforcing uniqueness only in application code.
- Holding DB transactions open across remote calls.
- Choosing serializable everywhere.
- Ignoring deadlock retries.
- Reading then writing without considering concurrent changes.

## Must Remember

- **Constraints are powerful correctness tools.**
- **Transactions group changes atomically.**
- **ACID isolation controls effects of concurrency.**
- **Read Committed is weaker than Serializable.**
- **Serializable simplifies reasoning but costs concurrency/abort risk.**
- **Keep transactions short.**
- **Do not hold DB locks while waiting on slow remote systems.**
- **Use atomic conditional updates/constraints for invariants.**
- **SQL is often the safest default when relationships and invariants matter.**

## Interview Revision Summary

For a correctness-sensitive operation ask:

```text
Invariant?
Transaction boundary?
Unique/check constraint?
Concurrent writers?
Isolation needed?
Atomic conditional update?
Lock/version?
What happens on deadlock/abort?
Any external dependency inside transaction?
```

## Explain Without Notes

Explain how you prevent overselling the last product unit when two requests arrive concurrently, using a relational database.

## Completion Checklist

- [ ] I understand keys/relationships/normalization/denormalization.
- [ ] I can explain ACID.
- [ ] I understand common isolation anomalies conceptually.
- [ ] I can compare Read Committed and Serializable.
- [ ] I understand deadlocks and retry.
- [ ] I use constraints/atomic updates for invariants.
