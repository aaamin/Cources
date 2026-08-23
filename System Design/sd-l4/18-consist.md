# Lesson 18 — Consistency, CAP & Quorums

**Phase:** Fundamentals  
**Session:** 18/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn strong/eventual/read-after-write expectations, correct CAP intuition, quorum concepts, conflicts, and how to choose consistency per operation.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Strong vs eventual

Strong consistency gives reads a single coherent committed view under the chosen model. Eventual consistency allows temporary disagreement that converges. One system can use different guarantees for different data.

## Read-after-write and monotonic reads

Read-after-write means a successful writer should not immediately observe an older version. Monotonic reads mean once a client sees version V, later reads should not go backward. These user-visible guarantees are often more useful than broad labels.

## CAP

CAP matters when a network partition prevents replicas from communicating. During that partition you cannot provide both unrestricted availability and strong consistency for the affected data. CAP is not a static shopping label like “database X is AP forever.”

## Quorums

With N replicas, writes may wait for W and reads query R. The common overlap intuition `R + W > N` increases the chance that reads see a recent write. Real systems add versioning, repair, and failure details.

## Conflicts

Multi-writer or eventually consistent systems can accept concurrent conflicting updates. Resolution may use timestamps, application merge rules, versions, or rejection. Business semantics decide what is safe.

## Per-operation consistency

A like count can tolerate seconds of staleness; user settings often want read-after-write; seat reservation needs strong correctness. State the guarantee for the operation, not the whole product.

## Worked example — three feature guarantees

Like count: eventual is acceptable. Profile settings: read-after-write is desirable. Seat reservation: two users must not both confirm one seat, so stronger coordination is required. This illustrates why consistency is a product decision.

## Interview lens

Avoid reciting CAP theory. Translate consistency into user-visible behavior during failures and concurrent writes.

## What to remember

Use the weakest consistency that still preserves required business behavior. Stronger coordination has latency/availability cost.

## Review after reading

1. Eventual consistency allows what?
2. Read-after-write?
3. When is CAP relevant?
4. Quorum intuition?
5. One stale-tolerant and one strict feature?

## Deeper study notes

### Consistency has scope and time

Ask “consistent with respect to what?” Per-key linearizable reads, per-session read-after-write, per-conversation ordering, and globally serializable transactions are very different guarantees and costs. Avoid using “strong consistency” without naming the operation/scope.

### Availability is also scoped

During a partition, a system can keep reads available from stale replicas while rejecting conflicting writes. It can keep one region writable and make another read-only. CAP does not require one all-or-nothing product-wide choice.

### Quorums are not magic

`R + W > N` is a useful overlap intuition but assumes replicas and versions are managed appropriately. Sloppy quorums, hinted handoff, concurrent writes, and failures add nuance. For interviews, use quorum language to explain the latency/consistency trade-off, not to claim mathematical perfection.

### Eventual consistency requires convergence

“Eventually consistent” is not permission to ignore conflicts. The system needs a mechanism that causes replicas/derived views to converge: last-write rule, version-based reconciliation, stream replay, anti-entropy, or application merge.

### Common mistakes

- Saying CAP means choose only two of C/A/P in normal operation.
- Demanding strong consistency for like counts or presence with no business reason.
- Using eventual consistency for money/inventory without explaining conflict handling.
- Assuming timestamps always resolve concurrent writes correctly despite clock skew.


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

**Next:** Lesson 19 — Message Queues & Async Processing
