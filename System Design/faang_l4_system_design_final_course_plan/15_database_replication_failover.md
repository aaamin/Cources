# Session 15 — Database Replication & Failover

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Understand why copies of data exist, how lag appears, and what failover does to correctness and availability.

## What You Need to Read / Learn

- Leader/follower replication.
- Read replicas and read scaling.
- Synchronous versus asynchronous replication.
- Replication lag and stale reads.
- Read-after-write anomalies when reads go to followers.
- Leader failure and failover.
- Split-brain concept and why only one writer may need to be authoritative.
- Replica promotion and potential data loss under asynchronous replication.
- Backups versus replicas: different purposes.

## What You Need to Do

- [ ] Explain a user updating profile data then immediately reading stale data from a replica.
- [ ] Design read routing for data that needs read-after-write consistency.
- [ ] Walk through what can happen if the leader dies immediately after acknowledging a write.

## **Must Remember for the Interview**

- **Replication improves availability/read capacity but does not automatically solve partitioning or correctness.**
- **Async replication can lose the newest acknowledged writes during failover.**
- **Followers can be stale. Route consistency-sensitive reads accordingly.**
- **A replica is not a backup against accidental deletion or logical corruption.**
- **Failover has a correctness story, not just an availability story.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Leader handles authoritative writes; followers replicate and can serve reads.**
- **Sync replication → stronger durability, higher write latency. Async → lower latency, lag/data-loss risk.**
- **Read-after-write may require leader reads/session pinning/version-aware routing.**
- **Failover must avoid split brain.**
- **Replication and backup solve different failure classes.**

## Self-Test Before Marking This Session Complete

- [ ] Can I explain replication lag?
- [ ] Can I explain synchronous vs asynchronous replication?
- [ ] Can I distinguish replica from backup?
- [ ] Can I explain a failover data-loss scenario?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 15/46  
**Next:** Session 16
