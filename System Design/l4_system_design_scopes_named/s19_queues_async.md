# Session 19 — Message Queues & Async Processing

## 1. Must Learn

### Queue model
- **Understand:** Understand producer → queue → consumer/worker and why queues decouple/buffer work.
- **Decision/trade-off:** Lower request latency/isolation vs eventual completion/operational complexity.

### Acknowledgement & visibility
- **Understand:** Know a message should remain/reappear if work is not safely acknowledged; visibility timeout conceptually.
- **Decision/trade-off:** Fast redelivery vs duplicate processing.

### Retries, poison messages & DLQ
- **Understand:** Understand transient retry vs repeatedly failing messages routed aside.
- **Decision/trade-off:** Reliability vs retry storms/backlog; isolate unrecoverable work.

### Delivery semantics
- **Understand:** Know at-most-once and at-least-once; do not casually promise end-to-end exactly-once.
- **Decision/trade-off:** Duplicate risk vs lost-work risk.

### Idempotent consumers
- **Understand:** Consumers should tolerate redelivery for side effects that must happen once logically.
- **Decision/trade-off:** Correct retries vs state/dedup complexity.

### Ordering guarantees
- **Understand:** Understand ordering may be global, per partition/key, or absent.
- **Decision/trade-off:** Ordering correctness vs parallelism/throughput.

### Backlog & worker scaling
- **Understand:** Queue depth/age indicates whether consumers keep up.
- **Decision/trade-off:** Elastic throughput vs downstream saturation.

## 2. Should Know

- Move non-user-blocking work off the synchronous path.
- Worker completion before ack is a key failure case.
- Backpressure: scaling consumers cannot exceed downstream capacity safely.

## 3. Recognition Only

- Exactly-once broker features vs true end-to-end semantics
- Priority queues

## 4. Important Comparisons

- Synchronous processing vs queued async work.
- At-most-once vs at-least-once.
- Retry vs DLQ.
- Global ordering vs partition/key ordering.
- More workers vs downstream capacity.

## 5. Important Interview Questions

1. What work can leave the synchronous request path?
2. What if a worker finishes but crashes before acknowledging?
3. How are duplicate messages handled?
4. What ordering scope is truly required?
5. What happens to poison messages?
6. What happens when queue backlog grows faster than workers can process it?

## 6. Common Interview Mistakes

- **Claiming exactly-once casually** → Prefer at-least-once + idempotent consumer when appropriate.
- **Acking before durable side effect** → Could lose work after crash.
- **Retry forever** → Bound retries and isolate poison messages.
- **Global ordering unnecessarily** → Use the smallest ordering scope required.
- **Autoscaling workers without downstream limits** → Respect dependency capacity/backpressure.

## 7. Communication

### Important Vocabulary

producer, consumer, worker, queue, acknowledgement, visibility timeout, retry, dead-letter queue, poison message, at-most-once, at-least-once, idempotent consumer, ordering, backlog

### Useful Interview Phrases

- “I’d use at-least-once delivery and make the consumer idempotent.”
- “The critical failure is work completing before the acknowledgement is persisted.”
- “I only need ordering per entity/partition, which preserves parallelism.”

### Important Questions to Ask the Interviewer

- **Question:** “Can this work complete asynchronously?”  
  **Why it matters:** Determines queue use.
- **Question:** “Can the side effect be safely repeated?”  
  **Why it matters:** Determines idempotency/dedup needs.
- **Question:** “What ordering scope is required?”  
  **Why it matters:** Controls partitioning and throughput.

## 8. ⭐ Must Remember

1. Queues decouple and buffer work.
2. At-least-once implies duplicates.
3. Idempotent consumers make retries safer.
4. Ack timing matters.
5. Poison messages need bounded retry/DLQ handling.
6. Ordering reduces available parallelism.
7. Backlog is a capacity/failure signal.

## 9. Study Priority

1. Study first: queue model, ack/visibility, delivery semantics.
2. Study next: idempotency, retries, DLQ.
3. Finish with: ordering, backlog, worker scaling.

## 10. Revision Checklist

- [ ] Choose sync vs async work.
- [ ] Explain at-most-once vs at-least-once.
- [ ] Handle worker crash-before-ack.
- [ ] Design duplicate-safe consumer behavior.
- [ ] Discuss ordering and backlog.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
