# FAANG / Big Tech L4 System Design Interview Course Plan — FINAL

## Final Course Structure

This is the final version of the course.

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
- Heavier fundamentals (especially Sessions 25–26): **90–120 minutes if needed**
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
- Main read/write flows
- Deep dives
- Bottlenecks
- Failure handling
- Trade-offs
- Closing summary
- Time management
- Interviewer collaboration

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
- Capacity headroom
- Growth assumptions

### Principle

Only calculate numbers that can influence a design decision.

### Target

For every important estimate, state the architectural implication.

Examples:

```text
High read QPS      → cache / replicas may matter
Large media volume → object storage + CDN
High write QPS     → partitioning / batching may matter
Many connections   → connection gateways / persistent connection capacity
```

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
- Connection reuse
- Latency vs throughput
- Bandwidth
- Request timeout
- Network hops
- Regional latency conceptually

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

Understand where latency and network failure can occur.

---

## Session 4 — Application Architecture, Service Boundaries & Request Lifecycle

### Learn

#### Architecture styles

- Monolith
- Modular monolith
- Microservices
- Service-oriented decomposition conceptually
- When to keep a system simple
- When to split a service
- When not to use microservices

#### Service boundaries

- Domain/service ownership
- Cohesion
- Coupling
- Independent deployment
- Independent scaling
- Database ownership
- Database-per-service concept
- Shared-database trade-offs

#### Communication

- Client/server model
- Application servers
- Stateful vs stateless services
- Service-to-service calls
- Synchronous communication
- Asynchronous communication
- Network/partial-failure cost of distributed services

#### Request lifecycle

- Request lifecycle
- Connection pools
- Timeouts
- Synchronous request chains
- Dependency latency
- Failure propagation

### Core trade-off

> Microservices can improve independent ownership, deployment, and scaling, but introduce network failures, distributed data, observability, deployment, and consistency complexity.

### Target

Given a system, explain whether you would start with:

```text
Monolith
Modular Monolith
Microservices
```

and defend the choice.

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
- Cross-zone / multi-AZ traffic conceptually
- Load-balancer failure considerations

### Target

Explain why horizontal scaling generally requires traffic distribution.

---

## Session 6 — Horizontal & Vertical Scaling

### Learn

- Scale up
- Scale out
- Stateless application servers
- Autoscaling concepts
- Scaling triggers
- CPU / memory / network limits
- Bottlenecks
- Session-state problems
- Shared state
- Single points of failure
- Capacity headroom
- Scaling limits of dependencies

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

Then explain what breaks next as traffic grows.

---

## Session 7 — Proxy, Reverse Proxy, API Gateway & Rate Limiting

### Learn

#### Proxy / gateway

- Forward proxy
- Reverse proxy
- API gateway
- Routing
- Authentication at the edge
- Request aggregation
- TLS termination
- Service discovery conceptually

#### Rate limiting

- Why rate limiting exists
- Fixed window
- Sliding window
- Token bucket
- Leaky bucket
- Per-user limits
- Per-IP limits
- Per-tenant limits
- Global limits
- Distributed counters conceptually
- Fail-open vs fail-closed
- Rate-limit response behavior

### Target

Explain the difference between:

- load balancer
- reverse proxy
- API gateway
- rate limiter

and where each belongs.

---

## Session 8 — CDN & Edge Caching

### Learn

- Origin server
- Edge location
- Static assets
- Media delivery
- Cache keys
- TTL
- Cache purge / invalidation
- Signed URLs conceptually
- Origin protection
- Cache-control concepts
- Dynamic vs static content
- Regional origin considerations

### Target

Know when CDN caching reduces:

- latency
- bandwidth
- origin load
- backend cost

and when content should not be cached publicly.

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
- Cache key design
- Source of truth
- Serialization/storage overhead

### Core questions

- What is the source of truth?
- What is the cache key?
- How long can data be stale?
- How is the cache invalidated?
- What happens on a cache miss?
- What happens when the cache is unavailable?

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
- Request coalescing conceptually
- Randomized TTLs
- Distributed cache failure
- Cache eviction under pressure
- Stale-while-revalidate conceptually

### Target

Explain how your system behaves when:

```text
Cache hit rate drops suddenly
Cache becomes unavailable
One key becomes extremely hot
Many keys expire at once
```

---

## Session 11 — SQL, Relational Databases & Transactions

### Learn

#### Relational model

- Tables
- Rows
- Relationships
- Primary keys
- Foreign keys
- Normalization
- Denormalization
- Constraints
- Joins

#### Transactions

- ACID
- Atomicity
- Consistency
- Isolation
- Durability
- Transaction boundaries
- Long-running transaction problems

#### Isolation

- Dirty reads
- Non-repeatable reads
- Phantom reads conceptually
- Read committed
- Repeatable read
- Serializable
- Isolation vs concurrency trade-offs
- Deadlocks
- Deadlock retry conceptually

### Target

Understand why relational databases are often the safest default when relationships and invariants matter.

Be able to explain when stronger isolation is worth lower concurrency.

---

## Session 12 — Database Indexing & Query Access Paths

### Learn

- Primary index
- Secondary index
- Composite index
- Covering index conceptually
- Index selectivity
- Query-driven indexing
- Index ordering
- Prefix behavior of composite indexes conceptually
- Read amplification
- Write amplification
- Index storage cost
- Full-text index conceptually
- Geospatial index conceptually
- When an external search index may be more appropriate

### Target

For a query:

1. identify the access pattern;
2. choose indexes;
3. explain why the index helps;
4. explain the write/storage cost.

---

## Session 13 — Data Modeling, Access Patterns & Read Models

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
What data can be precomputed?
```

### Also learn

- Normalized write models
- Denormalized read models
- Materialized/precomputed views conceptually
- Read-model duplication
- Derived data
- Ownership of source-of-truth data

### Target

Separate:

> “How should the data be modeled?”

from:

> “Which database product should I use?”

and from:

> “Which data should be precomputed for faster reads?”

---

## Session 14 — NoSQL & Specialized Datastores

### Learn

#### Core NoSQL families

- Key-value stores
- Document databases
- Wide-column databases
- Flexible schemas
- Horizontal scaling
- Query limitations
- Denormalization
- When joins become problematic

#### Specialized stores — recognition depth

- Search engines
- Inverted indexes conceptually
- Time-series databases
- Graph databases
- Geospatial storage/indexing conceptually

#### Selection principles

- Access-pattern-driven storage selection
- Operational complexity
- Consistency requirements
- Query flexibility
- Scale
- Cost
- Primary store vs secondary index/search system

### Target

Choose SQL vs NoSQL vs a specialized store based on workload and access patterns rather than popularity.

---

## Session 15 — Database Replication, Leaders & Failover

### Learn

#### Replication models

- Single-leader / leader-follower
- Multi-leader conceptually
- Leaderless conceptually

#### Replication behavior

- Read replicas
- Synchronous replication
- Asynchronous replication
- Replication lag
- Read-after-write issues
- Stale reads
- Replication conflicts
- Conflict-resolution conceptually

#### Failover

- Leader failure
- Leader election conceptually
- Promotion of a replica
- Failover
- Split-brain concept
- Availability during failover
- Recovery after failover

### Target

Explain what users might observe when reading from a lagging replica.

Be able to compare, at recognition level:

```text
Single leader
Multi leader
Leaderless
```

---

## Session 16 — Partitioning, Sharding & Distributed IDs

### Learn

#### Partitioning

- Horizontal partitioning
- Hash partitioning
- Range partitioning
- Geographic partitioning
- Partition keys
- Hot partitions
- Scatter-gather queries
- Rebalancing
- Resharding
- Tenant partitioning
- Time-based partitioning

#### Distributed ID generation

- Database auto-increment / sequence
- Central ID-generation service
- UUID
- Random IDs
- Snowflake-style / time-sortable IDs
- Timestamp + node + sequence intuition
- Globally unique vs locally unique IDs
- Ordering requirements
- Collision considerations
- Coordination requirements
- ID-generated hot spots
- Relationship between ID design and sharding

### Target

For every shard key, ask:

> What workload breaks this choice?

For every ID strategy, ask:

> Do I need uniqueness, ordering, decentralization, unpredictability, or shard-friendly distribution?

---

## Session 17 — Consistent Hashing

### Learn

- Hash ring intuition
- Virtual nodes conceptually
- Rebalancing
- Node addition/removal
- Distributed caches
- Partition ownership
- Uneven distribution
- Replication around a ring conceptually

### Target

Understand why consistent hashing can reduce data movement as nodes change.

Also understand that consistent hashing does not automatically solve:

- hot keys
- skewed workloads
- replication consistency
- cross-partition queries

---

## Session 18 — Consistency, CAP & Quorums

### Learn

- Strong consistency
- Eventual consistency
- Read-after-write consistency
- Monotonic reads conceptually
- Session consistency conceptually
- CAP during network partitions
- Read quorum
- Write quorum
- Quorum intuition
- Replication conflicts
- Availability trade-offs
- Stale reads
- Conflict resolution conceptually

### Target

For each important operation, explicitly state the required consistency.

Examples:

```text
Like count        → eventual consistency may be acceptable
Seat reservation  → strong correctness requirement
User profile read → often read-after-write is desirable
```

Avoid saying:

> “The whole system is strongly consistent.”

Instead define consistency per important operation/invariant.

---

## Session 19 — Message Queues, Delivery Semantics & Async Processing

### Learn

- Producer
- Consumer
- Worker
- Queue
- Acknowledgement
- Visibility timeout concept
- Retry
- Dead-letter queue
- Poison messages
- At-most-once
- At-least-once
- Duplicate processing
- Idempotent consumers
- Ordering guarantees
- Ordering scope
- Queue backlog
- Worker scaling

### Exactly-once principle

Understand that:

> End-to-end exactly-once processing is difficult and should not be casually promised.

A common practical design is:

```text
At-least-once delivery
        +
Idempotent consumer
```

### Target

Identify which operations can leave the synchronous request path.

Explain what happens if a worker completes work but crashes before acknowledging the message.

---

## Session 20 — Pub/Sub, Streams, Kafka Concepts & Reprocessing

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
- Consumer lag
- Partition skew
- Rebalancing consumers conceptually
- Batching
- Replay/reprocessing
- Backfill from an event stream
- Duplicate effects during replay

### Target

Know the difference between:

> “Perform this task once.”

and:

> “Publish an event that multiple consumers may independently process.”

Understand the implications of replaying historical events.

---

## Session 21 — Reliability, Overload & Failure Isolation

### Learn

- Timeouts
- Retry policies
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
- Dependency isolation
- Cascading failures
- Health checks
- Fail-fast behavior
- Capacity protection

### Target

Be able to answer:

> What happens when a dependency becomes slow rather than completely unavailable?

Be able to explain why retries can make an overloaded system worse.

---

## Session 22 — Idempotency, Concurrency, Distributed Coordination & Workflows

### Learn

#### Concurrency

- Race conditions
- Optimistic locking
- Pessimistic locking
- Compare-and-set
- Unique constraints
- Version numbers

#### Distributed coordination

- Leases
- Distributed locks conceptually
- Lease expiry
- Clock skew
- Stale lock holders
- Fencing tokens conceptually
- Leader election recognition
- Why a lock alone may not guarantee correctness

Example failure:

```text
Worker A gets lease
Worker A pauses
Lease expires
Worker B gets lease
Worker A wakes up
```

How do you stop Worker A from committing stale work?

#### Idempotency

- Idempotency keys
- Duplicate requests
- Duplicate events
- Idempotent consumers

#### Distributed workflows

- Saga pattern
- Compensation
- Transactional outbox
- CDC conceptually
- Reconciliation
- Dual-write problem
- CQRS/event sourcing at recognition depth only

### Target

Protect business invariants during:

- retries
- concurrent requests
- partial failures
- duplicate delivery
- stale workers

Example:

> Never charge a customer twice for one order.

---

## Session 23 — API, Authentication & Event Contract Design

### Learn

#### REST/API design

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

#### Other communication models

- gRPC concepts
- Webhooks
- Event contracts
- Schema evolution conceptually

#### Authentication recognition

- Cookie/session authentication
- Bearer tokens
- JWT conceptually
- API keys
- OAuth 2.0 conceptually
- OpenID Connect conceptually
- Token expiration
- Service-to-service authentication conceptually

### Target

Design APIs that make:

- retry behavior
- duplicate behavior
- pagination
- errors
- authentication expectations

explicit.

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
- Connection state
- Offline users
- Fan-out to connected clients
- Backpressure on realtime connections

### Target

Choose a communication model based on the workload rather than defaulting to WebSockets.

Be able to explain when:

```text
Polling
SSE
WebSocket
```

is the simplest appropriate choice.

---

## Session 25 — Object Storage, Media, Multi-Region & Disaster Recovery

### Object storage

Learn:

- Blob/object storage
- Metadata vs object data
- Multipart/resumable upload
- Checksums
- Presigned URLs
- Lifecycle policies
- Versioning
- Large-object handling

### Media

Learn conceptually:

- Upload pipeline
- Processing/transcoding
- Object storage
- CDN delivery
- Metadata storage
- Origin protection

### Multi-region

Learn:

- Single region
- Multi-AZ
- Active/passive
- Active/active
- Home-region models
- Global routing
- Cross-region replication
- Cross-region write conflicts
- Data residency
- Regional ownership of data
- Failover
- Latency vs consistency trade-offs

### Disaster recovery

Learn:

- Backup vs replica
- RPO
- RTO
- Restore testing
- Regional outage
- Backup retention
- Recovery validation

### Cost awareness

Learn:

- Storage cost
- Network egress
- Cross-region replication
- CDN economics
- Capacity headroom
- Cost of active/active architecture

### Target

Explain what happens when an entire region becomes unavailable.

Also explain why:

> A replica is not a backup.

---

## Session 26 — Observability, SLOs, Deployment, Migration, Security & Privacy

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
- Dependency latency

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
- Backward-compatible API/schema changes

### Schema evolution & data migration

- Expand-contract migration
- Backward-compatible schema rollout
- Data backfills
- Reprocessing
- Dual-read / dual-write migration recognition
- Rollback considerations
- Removing old schema safely

Typical sequence:

```text
Expand schema
    ↓
Deploy compatible application
    ↓
Backfill existing data
    ↓
Switch reads/writes
    ↓
Remove old schema later
```

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
- WAF awareness
- DDoS awareness
- Service-to-service trust boundaries

### Privacy & abuse

- PII
- Data retention
- Deletion
- Audit logs
- Rate limits
- Spam / scraping awareness
- Tenant isolation
- Abuse prevention

### Target

Be able to give a short production-readiness review covering:

```text
Observability
Reliability
Deployment
Data migration
Security
Privacy
Abuse
Cost
```

---

# Fundamental Phase Exit Gate

Before Session 27, you should be able to explain without notes:

## Interview & scale

- requirements and scope
- non-functional requirements
- back-of-the-envelope estimation
- peak traffic / skew / growth
- time management

## Networking & architecture

- DNS / TCP / TLS / HTTP
- request lifecycle
- load balancing
- reverse proxy
- API gateway
- rate limiting
- monolith
- modular monolith
- microservices
- service boundaries
- stateless services
- horizontal scaling

## Caching & edge

- CDN
- caching
- cache invalidation
- cache stampede
- hot keys
- cache failure

## Data

- SQL
- ACID
- transaction isolation
- indexes
- data modeling
- denormalization
- read models
- NoSQL families
- search/specialized stores
- replication
- failover
- leader models
- sharding
- distributed IDs
- consistent hashing
- consistency
- CAP
- quorums

## Async & reliability

- queues
- delivery semantics
- idempotent consumers
- Pub/Sub
- streams
- consumer groups
- ordering
- replay/reprocessing
- timeouts
- retries
- circuit breakers
- bulkheads
- backpressure
- load shedding

## Correctness

- race conditions
- optimistic/pessimistic locking
- unique constraints
- leases
- clock skew
- fencing
- idempotency
- Saga
- outbox
- CDC
- reconciliation

## APIs & realtime

- REST
- gRPC recognition
- pagination
- webhooks
- authentication mechanisms
- polling
- SSE
- WebSockets

## Production readiness

- object storage
- multi-region
- RPO/RTO/DR
- observability
- SLOs
- deployment
- schema migration/backfills
- security
- privacy
- abuse prevention
- cost

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
- Hot aliases

### Change request

Support custom aliases and prevent unsafe alias reuse.

---

## Session 28 — Design a Distributed Rate Limiter

### Main concepts

- Fixed window
- Sliding window
- Token bucket
- Leaky bucket
- Redis
- Atomic counters
- Sharding
- Per-user limits
- Per-tenant limits
- Global limits
- Failure policy
- Fail-open vs fail-closed
- Regional vs global enforcement

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
- Large objects
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
- Rate limiting
- Idempotency

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
- Reconnect/replay
- Partitioning

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
- Hot keys
- Precomputed read models

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
- Personalization
- Data pipeline

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
- Offline edits

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
- Retry/reprocessing

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
- Backpressure
- Partitioning

### Change request

Prioritize news freshness without overloading small websites.

---

# Phase 3 — Advanced System Designs

## Sessions 37–40

These designs intentionally emphasize correctness, high write rates, skew, failure recovery, and distributed coordination.

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
- Matching concurrency
- Regional failures

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
- Failure recovery

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
- Consumer lag
- Storage tiering

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
- Clock skew
- Fencing tokens
- Recurring jobs

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
Should this system start as a monolith or microservices?
How should chat messages be sharded?
Where should caching be used in Instagram?
Which notification operations should be asynchronous?
What consistency does ticket inventory need?
Which ID strategy fits this workload?
What happens when a lease holder becomes stale?
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
- authentication expectations
- error behavior
- pagination
- idempotency behavior
- entities
- ownership
- source of truth
- indexes
- retention
- consistency requirements

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

Do not split into many services without a reason.

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

### Scheduler

- leases
- stale workers
- retries
- duplicate execution

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
- schema migration
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

If **architecture boundaries** are weak:

- start with a modular monolith;
- identify which modules truly need independent scaling/deployment;
- compare the result with a microservice split.

If **correctness** is weak:

- inject duplicate requests;
- inject stale workers;
- inject concurrent updates;
- require an invariant to remain true.

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
- whether architecture decomposition/service boundaries are emphasized

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

## Infrastructure-oriented

- Distributed key-value store
- Configuration service
- Service discovery system
- Distributed lock/lease service
- Task queue

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
- interview-flow checklist
- architecture-style / service-boundary decision notes
- API design checklist
- authentication mechanism notes
- data modeling checklist
- SQL/NoSQL/specialized-store decision notes
- transaction/isolation notes
- indexing checklist
- replication/failover notes
- shard-key checklist
- distributed-ID decision notes
- cache decision template
- consistency decision matrix
- queue/event semantics checklist
- replay/reprocessing checklist
- idempotency/workflow checklist
- distributed-lock/lease/fencing notes
- reliability/failure checklist
- observability checklist
- security/privacy checklist
- multi-region/DR checklist
- migration/backfill checklist
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
- advanced cryptography
- Hadoop internals
- MapReduce internals
- vector database internals
- database storage-engine internals

Recognition-level knowledge is enough unless the target role specifically demands deeper distributed-systems expertise.

---

# Expected Progression

## Sessions 1–10

> I understand networking, architecture styles, service boundaries, scaling, traffic distribution, rate limiting, CDN, and caching.

## Sessions 11–18

> I understand transactions, indexing, data modeling, storage families, replication, partitioning, distributed IDs, and consistency.

## Sessions 19–26

> I understand asynchronous processing, delivery semantics, reliability, distributed coordination, APIs, realtime systems, workflows, multi-region systems, deployment/migrations, security, and observability.

## Sessions 27–36

> I can combine fundamentals into complete real systems.

## Sessions 37–40

> I can reason through high-scale, geo, contention, correctness, and failure-heavy problems.

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
Service Boundaries
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

When asked:

> **Should this be microservices?**

you should not automatically answer yes.

You should reason from:

```text
Domain boundaries
Independent scaling needs
Independent deployment needs
Team ownership
Data consistency
Operational complexity
Failure isolation
Expected system size
```

The goal is not to memorize architectures.

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

---

# Final Principle

> **Fundamentals first. Practice second. Repeated unseen performance determines readiness.**

This is the final company-neutral curriculum for **FAANG / Big Tech L4 system design interview preparation**.
