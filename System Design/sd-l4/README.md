# FAANG L4 System Design — Study Docs

This is the **documentation-first V2 course**. The lesson files are intended to be your primary study notes.

## Structure

```text
01–26  Fundamentals
27–36  Guided reference designs
37–40  Advanced reference designs
41–46  Timed unseen mocks
```

Recommended pace:

- fundamentals: 60–90 minutes;
- reference designs: 90–120 minutes;
- mocks: 45–55 minutes plus review.

If you miss a day, continue with the next unfinished lesson. Do not cram lessons just to catch up.

## How to study a fundamental lesson

1. Read the documentation end to end.
2. Re-read the worked example.
3. Close the file and explain the topic aloud for 3–5 minutes.
4. Use the review questions only afterward.
5. Record unclear points in the personal-notes section.

## How to study a reference design

1. Read how requirements lead to architecture.
2. Focus on the reusable pattern, not the final diagram.
3. Close the file.
4. Re-design the system yourself under a changed assumption.
5. Compare your reasoning with the reference.

## Readiness gate

Finishing lesson 46 is not sufficient by itself. You are ready when the **latest three unseen mocks** each score at least **32/40**, no category is below 2, and requirements, API/data, HLD, trade-offs, and communication average at least 3/4.

## Files

- [Lesson 1 — System Design Interview Framework](01-framework.md)
- [Lesson 02 — Back-of-the-Envelope Estimation](02-estimation.md)
- [Lesson 03 — Networking Fundamentals](03-networking.md)
- [Lesson 04 — Client–Server Architecture & Request Lifecycle](04-request.md)
- [Lesson 05 — Load Balancing](05-lb.md)
- [Lesson 06 — Horizontal & Vertical Scaling](06-scaling.md)
- [Lesson 07 — Proxy, Reverse Proxy & API Gateway](07-proxy.md)
- [Lesson 08 — CDN & Edge Caching](08-cdn.md)
- [Lesson 9 — Caching Fundamentals](09-cache.md)
- [Lesson 10 — Advanced Caching Problems](10-cache-adv.md)
- [Lesson 11 — SQL & Relational Databases](11-sql.md)
- [Lesson 12 — Database Indexing](12-index.md)
- [Lesson 13 — Data Modeling & Access Patterns](13-model.md)
- [Lesson 14 — NoSQL Databases](14-nosql.md)
- [Lesson 15 — Database Replication & Failover](15-repl.md)
- [Lesson 16 — Partitioning & Sharding](16-shard.md)
- [Lesson 17 — Consistent Hashing](17-chash.md)
- [Lesson 18 — Consistency, CAP & Quorums](18-consist.md)
- [Lesson 19 — Message Queues & Async Processing](19-queue.md)
- [Lesson 20 — Pub/Sub, Streams & Kafka Concepts](20-stream.md)
- [Lesson 21 — Reliability, Overload & Failure Isolation](21-reliability.md)
- [Lesson 22 — Idempotency, Transactions, Concurrency & Distributed Workflows](22-workflow.md)
- [Lesson 23 — API & Event Contract Design](23-api.md)
- [Lesson 24 — Real-Time Communication](24-realtime.md)
- [Lesson 25 — Object Storage, Media, Multi-Region & Disaster Recovery](25-storage-dr.md)
- [Lesson 26 — Observability, SLOs, Deployment, Security & Privacy](26-ops-sec.md)
- [Lesson 27 — Design a URL Shortener](27-url.md)
- [Lesson 28 — Design a Distributed Rate Limiter](28-rate.md)
- [Lesson 29 — Design Pastebin](29-paste.md)
- [Lesson 30 — Design a Notification Service](30-notify.md)
- [Lesson 31 — Design WhatsApp / Messenger](31-chat.md)
- [Lesson 32 — Design Twitter / Instagram News Feed](32-feed.md)
- [Lesson 33 — Design Search Autocomplete](33-auto.md)
- [Lesson 34 — Design Dropbox / Google Drive](34-drive.md)
- [Lesson 35 — Design YouTube / Video Platform](35-video.md)
- [Lesson 36 — Design a Distributed Web Crawler](36-crawl.md)
- [Lesson 37 — Design Uber / Ride Matching](37-uber.md)
- [Lesson 38 — Design Ticketmaster](38-tickets.md)
- [Lesson 39 — Design a Metrics / Logging Platform](39-metrics.md)
- [Lesson 40 — Design a Distributed Job Scheduler](40-jobs.md)
- [Lesson 41 — Mock #1 — Read-Heavy Service](41-mock-read.md)
- [Lesson 42 — Mock #2 — Realtime / Async Workflow](42-mock-rt.md)
- [Lesson 43 — Mock #3 — Data-Heavy / Write-Heavy](43-mock-write.md)
- [Lesson 44 — Mock #4 — High Scale with Skew](44-mock-skew.md)
- [Lesson 45 — Mock #5 — Correctness / Concurrency](45-mock-correct.md)
- [Lesson 46 — Final Unseen FAANG-Style Mock](46-final.md)

Also see `plan.md` and `score.md`.

## Core principle

> **Fundamentals first. Practice second. Repeated unseen performance determines readiness.**
