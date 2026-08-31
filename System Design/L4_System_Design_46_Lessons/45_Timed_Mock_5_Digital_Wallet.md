# Session 45 — Timed Mock #5 — Digital Wallet

## Timed Mock Instructions

**Time:** 45–55 minutes  
**Notes:** None  
**Solution lookup:** Not allowed  
**Goal:** Lead the interview aloud while drawing.

### Prompt

> Design a digital wallet where users can hold a balance and transfer money to another user.

### Your responsibilities

You must:
1. clarify functional requirements;
2. prioritize non-functional requirements;
3. estimate only design-relevant workload;
4. define APIs/events and core data;
5. draw a simple high-level architecture;
6. narrate one main read and one main write;
7. deep-dive into the hardest requirement;
8. respond to interviewer changes;
9. handle one failure scenario;
10. close with trade-offs and 10× evolution.

### Focus

Correctness, concurrency, idempotency, ledger modeling, external failure, audit/reconciliation.

### Information You May Assume Initially

- Wallet balances are denominated in one currency initially.
- Users can transfer wallet funds to another user.
- Transfer history is required.
- External bank/card top-up is not initially required.
- Negative balance is not allowed.

---

# STOP SCROLLING

Start your timer now.

Do **not** open the sections below until the appropriate point in the mock.

<details>
<summary><strong>INTERVIEWER PACK — Open only after your initial architecture</strong></summary>

### Clarifications
- Money must never be created or destroyed by an internal transfer.
- A transfer may be retried by the client after timeout.
- Users may send several transfers concurrently.
- Strong correctness is more important than maximum availability.

### Requirement Change
> Add wallet top-up through an external payment provider.

Ask:
- provider timeout/unknown outcome;
- idempotency;
- webhook duplicates;
- reconciliation.

### Failure Scenario
> The service commits the debit but crashes before the credit appears.

A correct design should make this impossible for a committed internal transfer or provide a transaction/ledger model that preserves the invariant.

### Concurrency Push
> Two outgoing transfers each try to spend the same remaining balance.

Ask how the invariant is enforced atomically.

</details>

<details>
<summary><strong>POST-MOCK REVIEW SIGNALS — Open only after the timer ends</strong></summary>

These are not a single canonical solution. Use them to detect whether you missed important reasoning.

Strong signals:

- immutable/double-entry-style ledger or at minimum an auditable transfer record;
- internal transfer debit+credit happen in one local DB transaction when wallets share transactional database/partition;
- authoritative balance can be derived/materialized from ledger with atomic update;
- row locking/conditional update/serializable strategy prevents overspend;
- idempotency key is unique and atomically claimed;
- client timeout returns/retrieves the same transfer, not a new transfer;
- transfer state machine and audit;
- external top-up uses provider idempotency key, webhook dedup, unknown/pending state, reconciliation;
- no long DB transaction held across external provider call;
- strong read/commit path for spending decisions; cache never authorizes a debit;
- partitioning by wallet makes cross-wallet transfer harder—candidate should acknowledge shard-transaction implications rather than casually shard early.

Critical warning signs:
- balance only in cache;
- read balance then write without lock/CAS;
- debit and credit separate async events with temporary money-loss semantics and no ledger invariant;
- “exactly once” without idempotency/reconciliation;
- external timeout treated as payment failure.


</details>

## 40-Point Scorecard

Score 0–4 in each category:

| Category | Score |
|---|---:|
| Requirements & Scope | /4 |
| Estimation & Workload | /4 |
| APIs / Events / Data Model | /4 |
| High-Level Design & Flows | /4 |
| Scalability & Performance | /4 |
| Correctness & Consistency | /4 |
| Reliability & Operations | /4 |
| Security / Privacy / Cost | /4 |
| Trade-Offs & Evolution | /4 |
| Communication & Time Control | /4 |
| **Total** | **/40** |

## Repair Rule

After scoring:

1. Identify the bottom two categories.
2. Write the single most damaging mistake in each.
3. Perform 2–3 narrow drills.
4. Redo only the weakest 15–20 minutes from a blank page.
5. Do not memorize a “perfect architecture.”

## Mock Completion Record

```text
Date:
Duration:
Score:
Bottom category #1:
Bottom category #2:
Best decision:
Biggest miss:
Requirement-change response:
Failure-scenario response:
What I will repair:
```
