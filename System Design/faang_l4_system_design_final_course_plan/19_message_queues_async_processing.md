# Session 19 — Message Queues & Async Processing

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Learn how queues decouple work, smooth bursts, and introduce delivery/retry semantics that applications must handle.

## What You Need to Read / Learn

- Producer, broker/queue, consumer/worker.
- Acknowledgements and visibility timeout intuition.
- At-most-once versus at-least-once delivery.
- Duplicate processing and idempotent consumers.
- Retries with backoff and jitter.
- Dead-letter queues and poison messages.
- Priority and fairness.
- Backpressure from growing queue depth.
- Ordering scope and why global ordering is expensive.
- When to move work off the synchronous request path.

## What You Need to Do

- [ ] Design an image-processing queue and explain worker crash before/after completion.
- [ ] Define retry and DLQ policy for notification delivery.
- [ ] Explain how the producer should behave when the queue is unavailable.

## **Must Remember for the Interview**

- **At-least-once delivery means duplicates are normal; consumers must tolerate them.**
- **A queue can absorb bursts only until backlog/retention/capacity limits are reached.**
- **Acknowledgement timing changes loss/duplicate behavior.**
- **Retries need backoff and a stopping policy.**
- **Ordering should be scoped to the smallest key that actually requires it.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Queue = decouple producer from worker + buffer bursts.**
- **Use idempotency for duplicate delivery.**
- **DLQ isolates poison work; it is not a substitute for diagnosis/replay.**
- **Monitor queue depth, oldest-message age, retry rate, and processing latency.**
- **Async work changes user-visible completion semantics.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain at-most-once vs at-least-once?
- [ ] Can I explain visibility timeout?
- [ ] Can I design a retry/DLQ policy?
- [ ] Can I explain when a queue is the wrong choice?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 19/46  
**Next:** Session 20
