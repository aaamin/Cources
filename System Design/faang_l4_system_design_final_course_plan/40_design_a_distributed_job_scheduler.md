# Session 40 — Design a Distributed Job Scheduler

**Phase:** Phase 3 — Advanced System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Practice ownership, leases, retries, duplicate execution, recurring jobs, priority, and worker recovery.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Job model: ID, schedule, payload/reference, state, retries, priority, tenant.
- Scheduler that finds due jobs.
- Queue/work dispatch.
- Worker leases/heartbeats.
- At-least-once execution and idempotent jobs.
- Retry/backoff/DLQ.
- Recurring scheduling and next-run computation.
- Stale worker/fencing problem.
- Long-running jobs and lease renewal.
- Priority/fairness.
- Result state and reconciliation.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design one-time jobs first, then recurring jobs.
- [ ] Change request: prevent a stale worker from committing after its lease expired.
- [ ] Walk through scheduler crash and worker crash scenarios.

## **Must Remember for the Interview**

- **Exactly-once execution is rarely a realistic transport guarantee; design idempotent business effects.**
- **Leases provide temporary ownership; lease expiry creates stale-worker risk.**
- **Fencing/version tokens can reject results from stale owners.**
- **Scheduler availability and worker execution are separate concerns.**
- **Recurring jobs need deterministic next-run behavior and protection against duplicate scheduling.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Due jobs → scheduler → durable queue → leased worker → result/update.**
- **Use job IDs/idempotency to tolerate redelivery.**
- **Heartbeat/renew lease for long jobs.**
- **Reject stale workers with version/fencing token where needed.**
- **Monitor queue age, schedule delay, retry rate, and stuck jobs.**

## Self-Test Before Marking This Session Complete

- [ ] Did I define job states?
- [ ] Did I handle duplicate execution?
- [ ] Did I handle stale workers?
- [ ] Did I design recurring scheduling?
- [ ] Did I explain scheduler/worker failure recovery?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support recurring jobs and prevent stale workers from committing results.

**Score using the 40-point rubric.**


---

**Progress:** Session 40/46  
**Next:** Session 41
