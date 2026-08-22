# FAANG L4 System Design Interview Course Plan — V2

## Final Course Structure

This is the finalized version of the course.

The design principle is:

> **Learn the fundamentals first. Then practice full systems. Then prove readiness through repeated unseen mocks.**

The course is **session-based, not calendar-based**.

- **26 Fundamentals Sessions**
- **10 Guided System Design Sessions**
- **4 Advanced System Design Sessions**
- **6 Timed Mock Interview Sessions**
- **46 Total Sessions**

Recommended workload:

- Fundamentals: **60–90 minutes**
- Guided designs: **90–120 minutes**
- Advanced designs: **90–120 minutes**
- Mock interviews: **45–55 minute mock + 30–45 minute review**

Do not skip sessions because you missed a day. Resume with the next unfinished session.

---

# Course Roadmap

| Phase | Sessions | Focus | Exit Goal |
|---|---:|---|---|
| Phase 1 | 1–26 | Complete system design fundamentals | Explain the main building blocks, trade-offs, and failure modes without notes |
| Phase 2 | 27–36 | Guided system design practice | Combine fundamentals into complete architectures |
| Phase 3 | 37–40 | Advanced system design practice | Handle correctness, high scale, skew, geo, and failure scenarios |
| Phase 4 | 41–46 | Unseen mock interviews | Demonstrate repeatable interview performance |

---

# Phase 1 — Fundamentals

## Sessions 1–26

No full system-design interview problems are required before Session 26.

Small examples and narrow design drills are encouraged, but the purpose of this phase is to build the underlying knowledge first.

---

## Session 1 — System Design Interview Framework

### Learn

- Functional requirements
- Non-functional requirements
- Scope and non-goals
- Back-of-the-envelope estimation
- APIs and events
- Data model
- High-level architecture
- Deep dives
- Bottlenecks
- Failure handling
- Trade-offs
- Closing summary

### Target

Be able to describe the interview flow:

```text
Requirements
    ↓
Scale
    ↓
APIs / Events
    ↓
Data Model
    ↓
High-Level Architecture
    ↓
Main Request Flows
    ↓
Deep Dive
    ↓
Scaling / Failures
    ↓
Trade-Offs
    ↓
Summary
```

---

## Session 2 — Back-of-the-Envelope Estimation

### Learn

- DAU / MAU
- Average and peak QPS
- Read/write ratio
- Payload size
- Storage growth
- Bandwidth
- Concurrent users/connections
- Retention
- Traffic skew
- Burst traffic

### Principle

Only calculate numbers that can influence a design decision.

---

## Session 3 — Networking Fundamentals

### Learn

- IP addresses
- DNS
- TCP
- UDP
- TLS
- HTTP / HTTPS
- Connections
- Keep-alive
- Latency vs throughput
- Request timeout

### Target

Trace a request from:

```text
Client
  ↓
DNS
  ↓
Network
  ↓
Load Balancer
  ↓
Application
  ↓
Database
  ↓
Response
```

---

## Session 4 — Client–Server Architecture & Request Lifecycle

### Learn

- Client/server model
- Application servers
- Stateful vs stateless services
- Request lifecycle
- Service-to-service calls
- Connection pools
- Timeouts
- Synchronous request chains

### Target

Understand where latency and failure can occur in a basic backend request.

---

## Session 5 — Load Balancing

### Learn

- L4 vs L7 load balancing conceptually
- Round robin
- Least connections
- Weighted routing
- Health checks
- Failover
- Connection draining
- Sticky sessions
- Global vs regional load balancing

### Target

Explain why horizontal scaling generally requires traffic distribution.

---

## Session 6 — Horizontal & Vertical Scaling

### Learn

- Scale up
- Scale out
- Stateless application servers
- Autoscaling concepts
- Bottlenecks
- CPU / memory / network limits
- Session-state problems
- Single points of failure

### Target

Be able to evolve:

```text
1 server
```

into:

```text
Load Balancer
      ↓
App Server Pool
      ↓
Shared Data Layer
```

---

## Session 7 — Proxy, Reverse Proxy & API Gateway

### Learn

- Forward proxy
- Reverse proxy
- API gateway
- Routing
- Authentication at the edge
- Rate limiting
- Request aggregation
- TLS termination
- Service discovery conceptually

### Target

Explain the difference between:

- load balancer
- reverse proxy
- API gateway

---

## Session 8 — CDN & Edge Caching

### Learn

- Origin server
- Edge location
- Static assets
- Media delivery
- Cache keys
- TTL
- Cache purge
- Signed URLs conceptually
- Origin protection

### Target

Know when CDN caching reduces latency, bandwidth, and backend load.

---

## Session 9 — Caching Fundamentals

### Learn

- Local cache
- Distributed cache
- Cache-aside
- Read-through
- Write-through
- Write-behind
- TTL
- Eviction
- Cache hit rate

### Core questions

- What is the source of truth?
- What is the cache key?
- How long can data be stale?
- How is the cache invalidated?

---

## Session 10 — Advanced Caching Problems

### Learn

- Cache invalidation
- Stale data
- Cache stampede
- Cache penetration
- Hot keys
- Cache warming
- Negative caching
- Thundering herd
- Distributed cache failure

### Target

Explain how your system behaves when the cache becomes unavailable.

---

## Session 11 — SQL & Relational Databases

### Learn

- Tables
- Rows
- Relationships
- Primary keys
- Foreign keys
- Normalization
- Denormalization
- Transactions
- Constraints
- Joins

### Target

Understand why relational databases are often the safest default when relationships and invariants matter.

---

## Session 12 — Database Indexing

### Learn

- Primary index
- Secondary index
- Composite index
- Covering index conceptually
- Index selectivity
- Read amplification
- Write amplification
- Query-driven indexing

### Target

For a query, identify which fields should be indexed and why.

---

## Session 13 — Data Modeling & Access Patterns

### Learn

Design schemas based on access patterns.

Example entities:

```text
User
Post
Comment
Like
Follow
```

Before choosing storage, ask:

```text
What are my primary reads?
What are my primary writes?
What must be unique?
What must be strongly consistent?
What needs ordering?
```

### Target

Separate:

> “How should the data be modeled?”

from:

> “Which database product should I use?”

---

## Session 14 — NoSQL Databases

### Learn

- Key-value stores
- Document databases
- Wide-column databases
- When joins become problematic
- Flexible schemas
- Horizontal scaling
- Query limitations
- Denormalization

### Target

Choose SQL vs NoSQL based on workload and access patterns rather than popularity.

---

## Session 15 — Database Replication & Failover

### Learn

- Leader/follower replication
- Read replicas
- Synchronous replication
- Asynchronous replication
- Replication lag
- Failover
- Split-brain concept
- Read-after-write issues

### Target

Explain what users might observe when reading from a lagging replica.

---

## Session 16 — Partitioning & Sharding

### Learn

- Horizontal partitioning
- Hash partitioning
- Range partitioning
- Geographic partitioning
- Partition keys
- Hot partitions
- Scatter-gather queries
- Rebalancing
- Resharding

### Target

For each shard key, ask:

> What workload breaks this choice?

---

## Session 17 — Consistent Hashing

### Learn

- Hash ring intuition
- Virtual nodes conceptually
- Rebalancing
- Node addition/removal
- Distributed caches
- Partition ownership

### Target

Understand why consistent hashing can reduce data movement as nodes change.

---

## Session 18 — Consistency, CAP & Quorums

### Learn

- Strong consistency
- Eventual consistency
- Read-after-write consistency
- Monotonic reads conceptually
- CAP during network partitions
- Read quorum
- Write quorum
- Replication conflicts
- Availability trade-offs

### Target

For each important operation, explicitly state the required consistency.

Examples:

```text
Like count        → eventual consistency may be acceptable
Seat reservation  → strong correctness requirement
User profile read → often read-after-write is desirable
```

---

## Session 19 — Message Queues & Async Processing

### Learn

- Producer
- Consumer
- Worker
- Queue
- Acknowledgement
- Visibility timeout concept
- Retry
- Dead-letter queue
- At-most-once
- At-least-once
- Duplicate processing

### Target

Identify which operations can leave the synchronous request path.

---

## Session 20 — Pub/Sub, Streams & Kafka Concepts

### Learn

- Queue vs Pub/Sub
- Event stream
- Topic
- Partition
- Offset
- Consumer group
- Retention
- Replay
- Ordering within a partition
- Log compaction conceptually
- Batch vs stream processing
- Aggregation windows conceptually

### Target

Know the difference between:

> “Perform this task once.”

and:

> “Publish an event that multiple consumers may independently process.”

---

## Session 21 — Reliability, Overload & Failure Isolation

### Learn

- Timeouts
- Retries
- Exponential backoff
- Jitter
- Circuit breakers
- Bulkheads
- Backpressure
- Admission control
- Load shedding
- Graceful degradation
- Retry storms
- Failure domains

### Target

Be able to answer:

> What happens when a dependency becomes slow rather than completely unavailable?

---

## Session 22 — Idempotency, Transactions, Concurrency & Distributed Workflows

### Learn

- Race conditions
- Optimistic locking
- Pessimistic locking
- Compare-and-set
- Unique constraints
- Leases
- Distributed locks conceptually
- Idempotency keys
- Duplicate events
- Saga pattern
- Compensation
- Transactional outbox
- CDC conceptually
- Reconciliation
- Dual-write problem
- CQRS/event sourcing at recognition depth only

### Target

Protect business invariants during retries and partial failures.

Example:

> Never charge a customer twice for one order.

---

## Session 23 — API & Event Contract Design

### Learn

- REST resources
- GET / POST / PUT / PATCH / DELETE
- Resource naming
- Pagination
- Cursor vs offset pagination
- Filtering
- Sorting
- API versioning
- Error handling
- Idempotency keys
- Rate-limit responses
- gRPC concepts
- Webhooks
- Event contracts

### Target

Design APIs that make retry and duplicate behavior explicit.

---

## Session 24 — Real-Time Communication

### Learn

- Polling
- Long polling
- Server-Sent Events
- WebSockets
- Persistent connections
- Heartbeats
- Connection gateways
- Presence
- Reconnection
- Session routing

### Target

Choose a communication model based on the workload rather than defaulting to WebSockets.

---

## Session 25 — Object Storage, Media, Multi-Region & Disaster Recovery

### Learn

#### Object storage

- Blob/object storage
- Metadata vs object data
- Multipart/resumable upload
- Checksums
- Presigned URLs
- Lifecycle policies
- Versioning

#### Multi-region

- Single region
- Multi-AZ
- Active/passive
- Active/active
- Home-region models
- Global routing
- Cross-region replication
- Failover

#### Disaster recovery

- Backup vs replica
- RPO
- RTO
- Restore testing
- Regional outage

#### Cost awareness

- Storage cost
- Network egress
- Cross-region replication
- CDN economics
- Capacity headroom

### Target

Explain what happens when an entire region becomes unavailable.

---

## Session 26 — Observability, SLOs, Deployment, Security & Privacy

### Observability

Learn:

- Metrics
- Logs
- Traces
- Correlation IDs
- Dashboards
- Alerts
- p50 / p95 / p99
- Error rate
- Throughput
- Queue depth
- Cache hit rate
- Saturation

### SLO concepts

- SLI
- SLO
- SLA
- Error budget conceptually

### Deployment

- Rolling deployment
- Canary deployment
- Blue/green deployment
- Rollback
- Backward-compatible API/schema change

### Security

- Authentication
- Authorization
- TLS
- Encryption at rest
- Encryption in transit
- Secrets
- Least privilege
- Signed URLs
- Input validation

### Privacy & abuse

- PII
- Data retention
- Deletion
- Audit logs
- Rate limits
- Spam / scraping awareness
- Tenant isolation

### Target

Be able to give a short production-readiness review of a design.

---

# Fundamental Phase Exit Gate

Before Session 27, you should be able to explain without notes:

- load balancing
- stateless services
- caching
- SQL vs NoSQL
- indexes
- replication
- sharding
- consistency
- queues
- Pub/Sub and streams
- retries
- backpressure
- idempotency
- Saga/outbox concepts
- REST APIs
- WebSockets
- object storage
- multi-region failover
- observability
- basic security

A concept is considered sufficiently learned when you can answer:

1. What problem does it solve?
2. How does it work conceptually?
3. When would you use it?
4. What are the trade-offs?
5. What happens when it fails?
6. What is one reasonable alternative?

---

# Phase 2 — Guided System Design Practice

## Sessions 27–36

These are full system-design problems, but they are still learning sessions.

For each session:

1. Attempt the design before studying a solution.
2. Use the interview framework.
3. Score the attempt.
4. Review reference material afterward.
5. Record the three highest-impact mistakes.
6. Redesign the weakest section.

---

## Session 27 — Design a URL Shortener

### Main concepts

- API design
- ID generation
- Alias uniqueness
- SQL vs NoSQL
- Read-heavy workload
- Caching
- Expiration
- Redirect latency
- Analytics separation

### Change request

Support custom aliases and prevent unsafe alias reuse.

---

## Session 28 — Design a Distributed Rate Limiter

### Main concepts

- Fixed window
- Sliding window
- Token bucket
- Redis
- Atomic counters
- Sharding
- Per-user limits
- Per-tenant limits
- Global limits
- Failure policy

### Change request

Support quotas across multiple regions.

---

## Session 29 — Design Pastebin

### Main concepts

- API
- ID generation
- Storage
- Expiration
- Read-heavy access
- Cache
- Abuse considerations

### Change request

Support very large pastes and configurable expiration.

---

## Session 30 — Design a Notification Service

### Main concepts

- Queue
- Workers
- Email / SMS / push providers
- Retries
- DLQ
- Deduplication
- Preferences
- Priority
- Fan-out
- Provider isolation

### Change request

Send an emergency campaign without delaying password-reset notifications.

---

## Session 31 — Design WhatsApp / Messenger

### Main concepts

- WebSockets
- Connection gateways
- Message storage
- Delivery semantics
- Message IDs
- Offline users
- Ordering
- Presence
- Multi-device synchronization

### Change request

Support very large group chats.

---

## Session 32 — Design Twitter / Instagram News Feed

### Main concepts

- Graph/follow relationships
- Feed generation
- Fan-out on write
- Fan-out on read
- Hybrid fan-out
- Feed cache
- Ranking
- Pagination
- Celebrity problem

### Change request

Support celebrity live posts and traffic spikes.

---

## Session 33 — Design Search Autocomplete

### Main concepts

- Prefix search
- Trie concepts
- Sorted indexes
- Search indexes
- Ranking
- Popular queries
- Caching
- Index updates
- Freshness

### Change request

Support trending queries and personalized suggestions.

---

## Session 34 — Design Dropbox / Google Drive

### Main concepts

- Object storage
- Metadata database
- Chunking
- Resumable uploads
- Checksums
- Versioning
- Synchronization
- Conflict resolution
- Sharing
- Deletion

### Change request

Support offline edits from multiple devices.

---

## Session 35 — Design YouTube / Video Platform

### Main concepts

- Upload
- Object storage
- Transcoding
- Processing queues
- Metadata
- Manifests
- CDN
- Adaptive streaming conceptually
- Popular content
- Origin protection

### Change request

A live event suddenly receives massive traffic.

---

## Session 36 — Design a Distributed Web Crawler

### Main concepts

- URL frontier
- Queue
- Workers
- Deduplication
- Canonicalization
- Robots/politeness
- Per-host limits
- Retry
- Storage
- Recrawl

### Change request

Prioritize news freshness without overloading small websites.

---

# Phase 3 — Advanced System Designs

## Sessions 37–40

These designs intentionally emphasize correctness, high write rates, skew, and failure recovery.

---

## Session 37 — Design Uber / Ride Matching

### Main concepts

- Driver location updates
- Geohash/grid/quadtree concepts
- Nearby search
- Candidate matching
- Ephemeral vs durable data
- WebSockets/events
- Regional partitioning
- Hot cities
- Stale location

### Change request

A stadium empties while regional connectivity is degraded.

---

## Session 38 — Design Ticketmaster

### Main concepts

- Inventory
- Seat holds
- Transactions
- Locking
- Expiration
- Payment
- Idempotency
- Saga
- Reconciliation
- Overselling
- Waiting room
- Audit trail

### Critical invariant

> Never sell the same seat twice.

### Change request

A global event sells out in seconds while the payment provider is slow.

---

## Session 39 — Design a Metrics / Logging Platform

### Main concepts

- High write throughput
- Event streams
- Partitioning
- Batching
- Time-series data
- Aggregation
- Retention
- Hot tenants
- Replay
- Query serving
- Backpressure

### Change request

One tenant suddenly produces 50× normal traffic.

---

## Session 40 — Design a Distributed Job Scheduler

### Main concepts

- Job queue
- Scheduler
- Workers
- Leases
- Heartbeats
- Retries
- Idempotency
- Priority
- Scheduling semantics
- Worker failure
- Duplicate execution
- Recovery

### Change request

Support recurring jobs and prevent stale workers from committing results.

---

# Phase 4 — Unseen Timed Mock Interviews

## Sessions 41–46

The final phase measures actual readiness.

These are not learning walkthroughs.

Rules:

- Prompt should be unseen.
- 45–55 minute design.
- No notes.
- No solution lookup.
- Explain while drawing.
- Include at least one interviewer-style requirement change.
- Include at least one failure scenario.
- Score every attempt using the rubric below.
- Repair weaknesses after each mock.

---

## Session 41 — Mock #1

Focus:

- Straightforward read-heavy or API-oriented service.

Goal:

Establish the first post-foundation mock baseline.

---

## Session 42 — Mock #2

Focus:

- Realtime, messaging, or asynchronous workflow.

Requirement change must be introduced after the initial architecture.

---

## Session 43 — Mock #3

Focus:

- Data-heavy or write-heavy system.

Must discuss:

- partitioning
- backpressure
- replay/recovery

---

## Session 44 — Mock #4

Focus:

- High-scale system with skew or hot keys.

Must explicitly identify:

> Which part breaks first at 10× traffic?

---

## Session 45 — Mock #5

Focus:

- Correctness / concurrency / commerce system.

Must handle:

- duplicate requests
- dependency timeout
- race condition
- business invariant

---

## Session 46 — Final Unseen Mock

Use the closest known format to the target company's actual system-design interview.

The interviewer should introduce:

1. an ambiguous requirement;
2. a mid-interview requirement change;
3. a failure scenario.

Afterward, produce a final readiness report.

---

# Standard Fundamental Session Format

Recommended duration: **60–90 minutes**

## 1. Closed-Book Recall — 5–10 min

Explain the previous session from memory.

## 2. Learn — 25–30 min

Study the required concept.

## 3. Trade-Offs & Failure Modes — 15–20 min

Ask:

- Why does this exist?
- When should I not use it?
- What are the main trade-offs?
- What breaks?
- How does it recover?

## 4. Small Application Drills — 15–20 min

Apply the concept to small scenarios.

Examples:

```text
How should chat messages be sharded?
Where should caching be used in Instagram?
Which notification operations should be asynchronous?
What consistency does ticket inventory need?
```

## 5. Explain Without Notes — 10 min

A session is incomplete if the concept cannot be explained clearly.

---

# Standard Full System Design Format

Use this from Session 27 onward.

## 0–6 min — Clarify Requirements

Identify:

### Functional requirements

What must the system do?

### Non-functional requirements

Prioritize:

- latency
- availability
- durability
- consistency
- throughput
- freshness
- cost

Define non-goals.

---

## 6–10 min — Estimate Scale

Estimate only useful numbers:

- DAU
- average QPS
- peak QPS
- read/write ratio
- storage
- bandwidth
- concurrent connections
- traffic skew

State the architectural implication of major estimates.

---

## 10–16 min — APIs / Events / Data Model

Define:

- APIs
- important events
- error behavior
- pagination
- idempotency behavior
- entities
- ownership
- source of truth
- indexes
- retention

Access patterns should drive the data model.

---

## 16–28 min — High-Level Architecture

Start simple.

Typical structure:

```text
Clients
   |
   v
Load Balancer / API Gateway
   |
   v
Application Services
   |
   +----> Cache
   |
   +----> Database
   |
   +----> Queue / Stream
   |
   +----> Object Storage
```

Narrate at least one main read flow and one main write flow.

---

## 28–42 min — Deep Dive

Choose the hardest requirement.

Examples:

### Messaging

- connection management
- delivery semantics
- ordering
- offline users

### Feed

- fan-out
- hot users
- ranking
- cache

### Ticket Booking

- holds
- concurrency
- payment timeout
- overselling

---

## 42–49 min — Production Review

Discuss relevant concerns:

- failure paths
- retries
- duplicate processing
- backpressure
- consistency
- security
- observability
- multi-region
- deployment
- cost

Do not add components merely to sound sophisticated.

---

## 49–52 min — Closing Summary

Summarize:

1. main architectural decisions;
2. important trade-offs;
3. biggest limitation;
4. how the system would evolve at 10× scale.

---

# 40-Point System Design Assessment Rubric

Score every serious design from Session 27 onward.

Each category receives **0–4 points**.

| Category | 0 | 2 — Developing | 4 — Strong |
|---|---|---|---|
| Requirements & Scope | Does not clarify | Finds main requirements but misses priorities | Controls ambiguity, priorities, constraints, and non-goals |
| Estimation & Workload | Missing or irrelevant | Reasonable estimates but weak connection to design | Estimates peak/skew/growth and ties numbers to decisions |
| APIs / Events / Data Model | Missing or incoherent | Happy path and main entities covered | Access-pattern driven, including errors, idempotency, ownership, indexes |
| High-Level Design & Flows | Incomplete architecture | Coherent happy path | Simple complete architecture with clear read/write/async flows |
| Scalability & Performance | No bottleneck reasoning | Uses common scaling components | Identifies workload-specific limits, hot spots, backpressure, 10× evolution |
| Correctness & Consistency | Ignored or contradictory | Names a consistency approach | Defines invariants, ordering, races, duplicates, and recovery precisely |
| Reliability & Operations | Ignores failure | Basic redundancy/monitoring | Covers failure domains, degradation, observability, deployment, restore |
| Security / Privacy / Cost | Ignored | Mentions common controls | Prioritizes meaningful threats, data lifecycle, abuse, and cost drivers |
| Trade-Offs & Evolution | Technology list only | Gives one alternative | Compares credible alternatives and evolves architecture only when justified |
| Communication & Time Control | Hard to follow | Understandable but reactive | Leads clearly, explains while drawing, incorporates changes, summarizes |

Maximum score:

```text
40 / 40
```

---

# Score Interpretation

| Score | Interpretation |
|---:|---|
| 34–40 | Strong L4-level performance for this prompt |
| 32–33 | Passing readiness signal |
| 28–31 | Close, but meaningful interview risk remains |
| 24–27 | Foundations exist; integration needs repair |
| Below 24 | Significant gaps remain |

A high total does **not** hide a critical weakness.

A score of **0 or 1** in any of these areas is considered a mock failure:

- Requirements
- APIs/Data
- High-Level Design
- Correctness
- Trade-Offs
- Communication

---

# Repair Rule

Do not respond to a weak mock by collecting random new design problems.

After every mock:

1. Score all 10 categories.
2. Identify the bottom 2 categories.
3. Perform 2–3 narrow drills for each weak category.
4. Redo the weakest 15–20 minutes of the old design from a blank page.
5. Attempt a fresh unseen mock later.
6. Compare category-level improvement.

Examples:

If **data modeling** is weak:

- model three small domains;
- list access patterns first;
- choose indexes;
- state consistency requirements.

If **reliability** is weak:

- take an old architecture;
- inject cache failure;
- inject database leader failure;
- inject dependency latency;
- inject regional outage.

---

# Final Readiness Gate

Finishing Session 46 is **not automatically readiness**.

You are considered system-design ready when all of the following are true:

1. All 26 fundamental sessions are complete.
2. All 14 practice designs were attempted.
3. At least 6 timed mock sessions were completed.
4. The **latest 3 unseen mocks each score at least 32/40**.
5. No category in those mocks scores below 2.
6. Requirements, APIs/Data, High-Level Design, Trade-Offs, and Communication average at least 3 across the latest three mocks.
7. You finish within the target interview time.
8. You can handle a requirement change without losing the structure of the design.
9. You can handle a failure scenario and explain recovery.
10. You can give a coherent 2-minute closing summary.

If the readiness gate is missed:

> **Repair the weakest category and attempt another unseen mock.**

Do not restart the entire course.

---

# Company Calibration

The base curriculum is intentionally company-neutral.

When a real interview is scheduled, confirm:

- exact interview duration
- HLD vs LLD expectations
- whether the round is product design or infrastructure design
- expected depth
- diagramming tool
- interviewer collaboration style
- target domain
- whether capacity estimation is expected
- whether API/data modeling is emphasized

Use this information to adjust Sessions 43–46.

Do **not** redesign the entire curriculum for each company.

The fundamentals remain the same.

---

# Practice Prompt Bank for Unseen Mocks

Do not study solutions for these before using them as a mock.

## Read-heavy / discovery

- Product catalog and filtering
- Restaurant discovery
- Trending topics
- App store search
- Advertisement serving read path

## Write-heavy / aggregation

- IoT telemetry platform
- Audit event platform
- Ad-click aggregation
- Distributed counter
- Metrics ingestion

## Realtime / collaboration

- Live sports comments
- Collaborative document editing
- Calendar scheduling and reminders
- Multiplayer presence
- Live auction

## Correctness / workflow

- Hotel reservation
- Payment workflow
- Digital wallet
- E-commerce checkout
- Coupon redemption

## Media / geo / miscellaneous

- Photo sharing
- Food delivery dispatch
- Podcast processing
- Map tile service
- Cloud build/job scheduler

Remove a prompt from the unseen pool after using it.

---

# Progress Tracking

Track:

```text
Sessions Completed: X / 46
```

Example:

```text
Day 1   Session 1 ✓
Day 2   Session 2 ✓
Day 3   Missed
Day 4   Session 3 ✓
Day 5   Session 4 ✓
```

Do not do:

```text
Day 4 → Session 3 + Session 4 + Session 5
```

The objective is:

> **46 high-quality completed sessions, not 46 consecutive study days.**

---

# Required Course Artifacts

By the end of the course, maintain:

- estimation cheat sheet
- API design checklist
- data modeling checklist
- SQL/NoSQL decision notes
- indexing checklist
- shard-key checklist
- cache decision template
- consistency decision matrix
- queue/event semantics checklist
- idempotency/workflow checklist
- reliability/failure checklist
- observability checklist
- security/privacy checklist
- multi-region/DR checklist
- 14 design diagrams and review notes
- 6 mock scores
- weakness/repair log
- final readiness report

Keep these concise.

The goal is not to build a textbook.

---

# What Not to Prioritize Before Completing This Course

Do not spend major preparation time on:

- Paxos implementation
- Raft implementation
- Byzantine consensus
- CRDT internals
- Kubernetes internals
- database engine implementation
- LSM-tree implementation
- B-tree implementation
- advanced service mesh internals
- advanced distributed transaction protocols
- sophisticated global consensus systems

Recognition-level knowledge is enough unless the target role specifically demands deeper distributed-systems expertise.

---

# Expected Progression

## Sessions 1–10

> I understand networking, scaling, traffic distribution, and caching.

## Sessions 11–18

> I understand data modeling, storage, replication, partitioning, and consistency.

## Sessions 19–26

> I understand asynchronous processing, reliability, APIs, realtime systems, workflows, multi-region systems, security, and observability.

## Sessions 27–36

> I can combine fundamentals into complete real systems.

## Sessions 37–40

> I can reason through high-scale, geo, contention, and failure-heavy problems.

## Sessions 41–46

> I can perform under real interview constraints.

---

# Final Readiness Target

When asked:

> **Design WhatsApp**

you should naturally reason through:

```text
Requirements
    ↓
Scale
    ↓
APIs / Events
    ↓
Data Model
    ↓
Connection Management
    ↓
Message Routing
    ↓
Message Storage
    ↓
Delivery Semantics
    ↓
Offline Users
    ↓
Partitioning
    ↓
Ordering
    ↓
Retries / Idempotency
    ↓
Failures
    ↓
Observability
    ↓
Scaling
    ↓
Trade-Offs
```

The goal is not to memorize WhatsApp.

The goal is to derive a reasonable design from requirements and defend every major architectural decision.

---

# Final Course Summary

| Phase | Sessions |
|---|---:|
| Fundamentals | 26 |
| Guided System Designs | 10 |
| Advanced System Designs | 4 |
| Timed Mock Interviews | 6 |
| **Total** | **46** |

## Final Principle

> **Fundamentals first. Practice second. Repeated unseen performance determines readiness.**

This is the final V2 curriculum for FAANG / Big Tech L4 system design interview preparation.
