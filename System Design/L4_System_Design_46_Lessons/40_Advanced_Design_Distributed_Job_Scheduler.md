# Session 40 — Advanced Design — Distributed Job Scheduler

## Interview Prompt

> Design a distributed job scheduler supporting one-time and recurring jobs, priorities, retries, worker failure, and recovery.

### Change Request

> Prevent stale workers from committing results after their lease expires and another worker has taken over.

This is a coordination/semantics interview.

---

# STOP — Complete Your Design First

Define:
- job states;
- scheduling/index structure;
- claiming;
- worker leases;
- heartbeat;
- retry;
- duplicate execution;
- idempotency;
- recurring job semantics;
- stale-worker fencing.

---

# Interviewer Pressure Pack

### Pressure 1
Worker A claims a 10-minute job with a 30-second lease. Worker A pauses for 60 seconds. Worker B then takes over. A resumes.

### Pressure 2
Scheduler crashes after putting a job on queue but before marking it dispatched.

### Pressure 3
A recurring job should run every day at 09:00. A region is unavailable from 08:50 until 10:30.

Should the missed run execute?

### Pressure 4
A job performs an external side effect and crashes before marking itself complete.

Respond first.

---

# Reference Reasoning

## 1. Job Model

```text
Job
- job_id
- type
- payload_ref
- schedule_at
- priority
- state
- attempt
- max_attempts
- lease_owner
- lease_expires_at
- fencing_token
- idempotency/business_key
- recurrence_spec?
- created_at
```

States:

```text
SCHEDULED
READY
RUNNING
SUCCEEDED
FAILED
CANCELLED
```

Retry may move:
```text
RUNNING → SCHEDULED(next_retry_at)
```

## 2. Separate Scheduler and Worker

```text
Job DB / Time Index
      ↓
Scheduler
      ↓
Ready Queue
      ↓
Workers
```

Scheduler determines which jobs are due.
Queue distributes ready work.
Workers execute.

Do not scan entire jobs table every second.

## 3. Finding Due Jobs

Options:
- DB index `(state, schedule_at)`;
- time buckets;
- timing wheel recognition;
- partitioned sorted sets/queues.

At moderate scale:
```sql
WHERE state='SCHEDULED'
  AND schedule_at <= now
ORDER BY schedule_at
LIMIT ...
```
with index and partitioned scheduler can work.

At huge scale:
- partition by time bucket + hash/job group;
- schedulers own partitions.

## 4. Dispatch Correctness

Problem:
scheduler selects job, publishes queue message, crashes before updating DB.

Possible duplicate dispatch on recovery.

This is acceptable if:
- job claim by worker is atomic/idempotent.

Alternative:
- outbox pattern: transactionally mark READY + outbox queue event.
- publisher delivers.

Do not require scheduler-to-queue exactly-once.

## 5. Worker Claim

Queue message says:
```text
job_id=123
```

Worker atomically claims:

```text
READY/SCHEDULED
  → RUNNING
  owner=A
  lease_expiry=...
  fencing_token=next
```

Conditional update ensures only one current owner.

Duplicate queue message:
- second worker fails claim if valid lease active.

## 6. Lease

Worker renews before expiry.

If worker crashes:
- no renew;
- lease expires;
- recovery process returns job to READY;
- another worker claims.

Lease prevents permanent stuck job.

But lease alone does not stop stale worker.

## 7. Stale Worker Problem

Timeline:

```text
A gets token 41, lease until :30
A pauses
lease expires
B gets token 42
B works
A resumes
```

If A writes result, it can overwrite B.

## 8. Fencing Token

Each successful claim gets monotonically increasing token.

Target resource/result store records highest token.

Write:
```text
job_id=123
token=41
```

After B writes with 42, A's 41 is rejected.

This works only if the side-effect destination participates in fencing/conditional version check.

If destination is an external API with no fencing support, you still need business idempotency/dedup strategy.

## 9. Heartbeats

Long job:
- renew lease;
- report progress.

Heartbeat failure could mean:
- worker dead;
- network partition;
- scheduler/store slow.

Do not instantly assume worker stopped executing.

That is why fencing is critical after reassignment.

## 10. Job Idempotency

At-least-once execution is common.

Job should be safe to repeat.

Examples:
- write output to deterministic object key/version;
- unique business record;
- provider idempotency key;
- check state transition.

If side effect is email, duplicate may still happen unless notification layer dedupes.

## 11. Retry

Classify errors:

Transient:
- dependency timeout;
- rate limit;
- temporary DB error.

Retry with:
- backoff;
- jitter;
- bounded attempts.

Permanent:
- invalid payload;
- missing required entity.

Fail/DLQ.

Avoid retry storm after global outage.

## 12. Priority

Options:
- separate queues;
- priority queues;
- weighted worker pools.

Prevent starvation:
- low priority should eventually progress.

Reserved capacity for critical jobs.

## 13. Recurring Jobs

Do not model recurrence as one permanently mutating job only.

Option:
- recurrence definition generates immutable occurrences.

```text
Schedule S
  ↓
Occurrence 2026-09-01 09:00
Occurrence 2026-09-02 09:00
```

Each occurrence has unique execution key:
```text
(schedule_id, intended_fire_time)
```

This makes retry/dedup/audit clearer.

## 14. Missed Run Semantics

Region down from 08:50–10:30 for 09:00 job.

Product policy choices:

### Catch-up
Run missed occurrence when recovered.

Good for:
- billing;
- daily report.

### Skip missed
Next occurrence only.

Good for:
- frequent refresh where old work has no value.

### Coalesce
If five intervals missed, run once with latest/current data.

Good for:
- cache refresh.

Need define **misfire policy** per schedule.

## 15. Time Zones / DST

Recurring “09:00 local time” differs from “every 24 hours.”

Need:
- schedule timezone;
- DST behavior;
- intended occurrence timestamp.

Do not implement cron parser in interview; recognize ambiguity.

## 16. Clock

Scheduler should use trusted server/database/coordination time.

Clock skew affects:
- due detection;
- lease expiry;
- recurrence.

Use:
- DB time;
- monotonic clocks for local durations where appropriate;
- safety margin.

Correctness must not depend blindly on worker wall clock.

## 17. Partitioning Scheduler

Shard jobs by:
- hash(schedule/job ID);
- time bucket + hash;
- tenant.

Each scheduler owns partitions via lease/leader election.

Hot tenant:
- dedicated partition/quotas.

Rebalance ownership when scheduler instance fails.

## 18. External Side Effect Crash

Job:
1. charges external provider;
2. crash before mark success.

Retry will run again.

Need:
- provider idempotency key = job/business occurrence ID;
- reconciliation if outcome unknown.

Same distributed workflow principles apply.

## 19. Cancellation

Cancel scheduled:
```text
SCHEDULED → CANCELLED
```

If already RUNNING:
- cooperative cancellation signal;
- may be too late to undo side effect.

Define semantics:
> cancellation prevents future work when possible; does not guarantee rollback of already-completed side effect.

## 20. Observability

Metrics:
- scheduled due lag;
- ready queue age;
- running jobs;
- lease expirations;
- duplicate claim conflicts;
- stale-token rejects;
- retry rate;
- permanent failures;
- duration p95/p99 by job type;
- scheduler partition ownership;
- misfires.

## Common Mistakes

- One cron server as SPOF.
- Full table scan every second.
- Lease without fencing.
- Exactly-once execution promised.
- Queue delivery treated as job ownership.
- No idempotency for external side effects.
- Recurrence ambiguity after downtime.
- Wall clock trusted blindly.
- Infinite retries.
- Cancellation assumed to roll back work.

## Must Remember

- **Scheduler identifies due work; queue distributes; worker claims authoritatively.**
- **Duplicate dispatch is acceptable if claims/idempotency are safe.**
- **Lease enables recovery after worker death.**
- **Lease expiry creates stale-worker risk.**
- **Fencing token prevents older ownership from committing where destination supports it.**
- **Jobs should assume at-least-once execution.**
- **Recurring schedules need explicit misfire/timezone semantics.**
- **External side effects need idempotency/reconciliation.**
- **Observe scheduling lag, not only queue depth.**

## Repair Exercise

Solve this precisely:

> Worker A with fencing token 101 writes a 5GB output. Its lease expires mid-upload. Worker B receives token 102 and successfully finishes. A later resumes and tries to mark output current.

Design object naming + metadata commit so A cannot replace B's result.
