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


## Detailed reference design

### Model scheduling and execution separately

A job definition says **what/when**. An execution says **this scheduled occurrence is running/finished**.

```text
Job
  schedule / payload / policy

Execution
  job_id
  scheduled_for
  attempt
  lease_owner
  lease_until
  state
```

This separation makes recurring jobs and retries much easier to reason about.

### Durable scheduler

Do not store millions of timers only in memory. Persist `next_run_at` and index/bucket by due time.

Schedulers query a near-future window:

```text
WHERE next_run_at <= now + lookahead
```

then atomically claim/create logical executions and enqueue them.

### Unique logical execution

For a recurring job, define identity:

```text
(job_id, scheduled_for)
```

with a unique constraint. If two scheduler instances race, only one logical execution record is created. This removes need for one global scheduler lock.

### Queue + workers

Due execution goes to ready queue. Worker claims it with lease, runs task, records result, then acknowledges.

At-least-once execution is realistic: a worker can finish external effect and crash before success record/ack.

### Leases

Execution has `lease_until`. Worker heartbeats extend it. If heartbeat stops, scheduler/reaper makes execution retryable after expiration.

A paused old worker can resume after lease loss. Include execution version/fencing token so authoritative result store rejects stale completion.

### Retry policy per job

Store:

- max attempts;
- timeout;
- exponential backoff;
- retryable error categories;
- dead-letter/final-failure behavior.

Not every job should retry identically. A daily report can retry for hours; a “send push before event starts” has a deadline.

### Recurrence semantics

For cron-like schedules, define behavior when system was down:

- run every missed occurrence;
- run only latest;
- skip missed;
- catch up with bounded rate.

Also define timezone/daylight-saving semantics. Store schedules in a canonical timezone and record intended occurrence time explicitly.

### Priority/fairness

Separate critical jobs from bulk work, or use weighted queues. A tenant scheduling millions of jobs should not starve other tenants. Per-tenant quotas and fair scheduling matter.

### Long-running jobs

Use heartbeats/progress checkpoints. A 6-hour job should not restart from zero after every worker crash if it can checkpoint safely.

## Failure walkthrough

### Scheduler instance dies

No problem if schedule state is durable and scheduler instances race using conditional claim/unique execution rows. Another instance continues.

### Queue duplicates

Workers dedupe by execution ID and business idempotency. Do not assume broker exactly-once.

### Worker completes after lease expired

Fencing/version rejects stale completion if a new worker owns the execution. External side effects still need idempotency because fencing your DB cannot undo an email/payment already sent.

### Top-of-hour cron burst

Bucket/shard due-time index, spread jitter when semantics allow, and scale queues/workers. Do not query one giant table with every job scheduled at `00:00` and no index.

## Interviewer follow-ups

### “How many schedulers?”

Multiple active scheduler instances are fine if due-job claiming is atomic/partitioned. Avoid one leader unless simplicity and availability requirements justify it.

### “How do you guarantee exactly once?”

I do not promise exact execution. I guarantee unique logical execution records and at-least-once worker attempts, then require idempotent effects or reconciliation.

### “What about dependency rate limits?”

Job queues can be partitioned/rate-limited by destination. Workers honor provider quotas and backoff so scheduled bursts do not overload dependencies.

### “How do users cancel a job?”

Update durable job/execution state. Workers check cancellation/version before side effects when practical. If already executing, cancellation semantics must be defined—best effort vs guaranteed before next checkpoint.

## Common interview mistakes

- One in-memory scheduler.
- Global lock for all due jobs.
- No unique logical occurrence ID.
- Lease with no stale-worker/fencing thought.
- Blind retry of non-idempotent effect.
- Recurring schedule with no missed-run semantics.
- No top-of-hour burst handling.
- No tenant fairness.

## Short revision note

**Scheduler pattern:** durable due-time index → unique execution record → ready queue → leased workers → idempotent side effects → retry/backoff → recurring next occurrence.

## Topics to revise

- [ ] Job vs Execution
- [ ] due-time index/bucket
- [ ] atomic claim
- [ ] execution identity
- [ ] lease/heartbeat
- [ ] fencing/idempotency
- [ ] retry policy
- [ ] recurrence/missed run
- [ ] priority/fairness

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll persist schedules and executions, not rely on in-memory timers. Scheduler instances atomically create unique logical executions for due jobs, enqueue them, and workers run under leases. At-least-once attempts are made safe through idempotency, execution IDs, and fencing/reconciliation.

## How the design evolves at 10×

At 10× jobs, shard/bucket due-time indexes and ready queues, isolate tenants/priorities, and avoid synchronized top-of-hour work with jitter where semantics allow. Long-running jobs need checkpoints and scalable heartbeat handling.

## Quick revision flashcards

**Logical execution ID?**  
Typically `(job_id, scheduled_for)` to dedupe scheduler races.

**Worker crash?**  
Lease expires and execution retries.

**Exactly once?**  
Not guaranteed for external side effects; idempotency/reconciliation creates safe business effect.

**Missed cron runs?**  
Product policy: catch up, latest only, or skip.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 41 — Mock #1 — Read-Heavy Service
