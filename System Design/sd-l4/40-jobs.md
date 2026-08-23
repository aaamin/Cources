# Lesson 40 — Design a Distributed Job Scheduler

**Phase:** Advanced Design  
**Session:** 40/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Schedule one-time and recurring jobs.
- Distribute due jobs to workers.
- Retry failures.
- Prevent duplicate business effects.
- Recover from scheduler and worker crashes.

## 2. Scale and workload shape

Estimate stored jobs, due jobs/sec, job duration distribution, retry rate, worker capacity, and burst patterns. Cron-like schedules often create synchronized top-of-minute/top-of-hour spikes.

## 3. API / contract surface

```http
POST /v1/jobs
GET  /v1/jobs/{id}
DELETE /v1/jobs/{id}
```

Job configuration includes schedule, payload reference, priority, retry policy, timeout, and optional idempotency information.

## 4. Data model

```text
Job(job_id, schedule, next_run_at, state, version, ...)
Execution(exec_id, job_id, scheduled_for, state, lease_until, attempt)
```

A unique logical execution ID can be derived from `(job_id, scheduled_for)` for recurring jobs.

## 5. High-level architecture

```text
Job API → Job DB
             ↓
        Scheduler Pool
             ↓
          Ready Queue
             ↓
          Workers
             ↓
     Execution State / Result
```

Schedulers claim due jobs with transactions/leases so several scheduler instances can run without enqueueing the same logical execution uncontrollably.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Recurring scheduling

Persist the schedule and next occurrence. After creating a durable execution, advance `next_run_at` atomically or idempotently. Missed schedules need an explicit catch-up policy.

### Worker lease

A worker claims an execution for a bounded time and renews while running. If it dies, another worker can retry after expiry.

### Stale workers and fencing

A slow worker can finish after its lease expired and a new worker took over. Business writes must be idempotent or verify execution/fencing version before commit.

### Priority and fairness

Separate queues or weighted scheduling prevent millions of low-priority jobs from starving urgent work.

## 7. Failure modes and recovery

- Scheduler crashes: another instance continues from durable job DB.
- Duplicate enqueue: unique execution ID/dedupe.
- Worker crash: lease expires and retry occurs.
- External side effect succeeds but worker crashes: idempotency key/reconciliation.
- Top-of-hour burst: sharded time buckets, prefetch, admission controls.
- Poison job: retry cap → DLQ/manual inspection.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

At-least-once execution is realistic. Exactly-once **business effect** comes from idempotent operations and durable execution state, not from trusting the queue to deliver once.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Scheduling and execution are separate responsibilities

The scheduler decides *when* a logical execution becomes ready. The worker decides *how* to run it. Separating them lets scheduler instances stay lightweight and lets worker pools scale by job type.

### Finding due jobs

Scanning a giant table every second is inefficient. Common patterns include time-bucketed partitions, an index on `next_run_at`, timing wheels, or loading near-future jobs into an in-memory scheduler while durable DB remains source of truth. Start with an indexed DB query and evolve only when scale demands it.

### Claiming due work safely

Multiple scheduler instances provide availability, but they must not all enqueue the same execution. Use transactional row claim, lease, version/CAS, or a unique execution record. Duplicates can still occur around crash boundaries, so downstream execution remains idempotent.

### Retry policy is part of the job definition

Jobs differ: a report can retry for hours; a payment reconciliation job may need strict frequency; an email can move to DLQ. Store max attempts, backoff, timeout, and perhaps retryable error categories.

### Exactly-once is usually the wrong promise

A worker can complete an external side effect and crash before recording success. The scheduler cannot know whether to retry without risking duplicate effect. Use idempotency keys at external APIs or reconciliation. Model the scheduler as at-least-once unless the effect system provides stronger transactional integration.

### Common interview mistakes

- One in-memory scheduler as source of truth.
- Global lock around all jobs.
- No lease expiry for crashed workers.
- Re-running non-idempotent jobs blindly.
- Storing millions of OS timers instead of durable scheduled state.
- Ignoring synchronized cron bursts.

### Reusable patterns learned

Durable schedule state, indexed due-time lookup, leases, unique logical executions, idempotent workers, fencing, priority isolation, and recurrence semantics.


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 41 — Mock #1 — Read-Heavy Service
