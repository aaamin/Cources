# Lesson 1 — System Design Interview Framework

**Course:** FAANG L4 System Design Interview Course Plan — V2  
**Session:** 1 / 46  
**Phase:** Fundamentals  
**Recommended study time:** 60–90 minutes

---

# 1. What You Will Learn

This lesson introduces the structure of a system design interview.

After reading this lesson, you should understand:

- what a system design interview is trying to evaluate;
- how to turn an ambiguous prompt into a structured problem;
- how to move from requirements to architecture;
- why estimation, APIs, data modeling, scaling, and trade-offs matter;
- how to manage a 45–60 minute interview;
- how to avoid common beginner mistakes.

The framework introduced here will be reused throughout the rest of the course.

The core sequence is:

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
Scaling & Failure Handling
    ↓
Trade-Offs
    ↓
Summary
```

Do not try to memorize this as a rigid script.

The purpose is to give you a reliable mental model so that, when the interviewer gives you an unfamiliar problem, you know what to do next.

---

# 2. What Is a System Design Interview?

A system design interview asks you to design a software system under a set of functional and technical constraints.

Typical prompts include:

```text
Design a URL shortener.
Design WhatsApp.
Design YouTube.
Design an online ticket-booking system.
Design a notification service.
```

The interviewer usually does **not** expect one perfect architecture.

The interview is primarily evaluating your engineering reasoning.

For an L4-level interview, you are generally expected to demonstrate that you can:

- clarify an ambiguous problem;
- identify the important requirements;
- estimate scale when it matters;
- define sensible APIs and data models;
- create a coherent architecture;
- recognize bottlenecks;
- make reasonable scalability decisions;
- handle failures;
- discuss trade-offs;
- communicate clearly.

A useful way to think about the interview is:

> You are not being asked, “Do you know the architecture of Twitter?”

You are being asked:

> “Given a problem and some constraints, can you reason your way toward a reasonable architecture?”

That distinction is extremely important.

---

# 3. The Wrong Way to Approach System Design

A common beginner approach is to hear a prompt and immediately start naming technologies.

Example:

```text
Interviewer:
Design Instagram.

Candidate:
We will use Kafka, Redis, Cassandra, Elasticsearch,
Kubernetes, microservices, and a CDN.
```

This sounds technical, but it does not show good design reasoning.

The interviewer still does not know:

- which requirements the system supports;
- why those technologies are needed;
- what the workload looks like;
- what data is stored;
- what consistency is required;
- how requests flow through the system;
- what the major trade-offs are.

Technology names are not architecture.

A stronger approach is:

```text
Problem
   ↓
Requirements
   ↓
Workload
   ↓
Required capabilities
   ↓
Architecture
   ↓
Technology choices
```

For example:

```text
Requirement:
Users frequently read the same product information.

Observation:
The workload is read-heavy and product data changes infrequently.

Capability needed:
A fast layer that reduces repeated database reads.

Design decision:
Introduce caching.

Possible implementation:
Redis or another distributed cache.
```

This demonstrates reasoning.

The general rule is:

> **Capability first, product second.**

---

# 4. Stage 1 — Clarify the Requirements

The first task in a system design interview is to understand what you are actually designing.

Most system design prompts are intentionally vague.

For example:

> Design a messaging system.

This could mean many different things.

Should it support:

- one-to-one messaging?
- group chats?
- voice calls?
- video calls?
- file sharing?
- message history?
- disappearing messages?
- read receipts?
- presence?
- end-to-end encryption?

Trying to design all of these would be unrealistic in a 45–60 minute interview.

So the first step is to define the scope.

---

# 5. Functional Requirements

Functional requirements describe **what the system must do**.

For a messaging system, a reasonable initial scope might be:

```text
1. A user can send a text message to another user.
2. A user can receive messages.
3. Users can view conversation history.
```

You could then ask whether group messaging is required.

A useful interview statement might be:

> “I’ll focus first on one-to-one text messaging, message history, and online/offline delivery. We can add group messaging if time permits.”

This shows that you can control scope.

---

# 6. Non-Functional Requirements

Non-functional requirements describe **how well the system must behave**.

Common non-functional requirements include:

- scalability;
- low latency;
- high availability;
- durability;
- consistency;
- security;
- fault tolerance;
- cost efficiency.

For a messaging system:

```text
Messages should normally be delivered with low latency.
Messages should not be lost once accepted.
The service should remain available during individual server failures.
The system should support a very large number of concurrent connections.
```

These requirements matter because they influence architecture.

For example:

```text
Low latency
    ↓
Avoid unnecessary synchronous network hops

High durability
    ↓
Persist messages before acknowledging success

Large connection count
    ↓
Use horizontally scalable connection servers
```

---

# 7. Define Non-Goals

Non-goals specify what you will intentionally leave out.

This is useful because interview time is limited.

Example:

```text
In scope:
- one-to-one text messages;
- message history;
- online/offline delivery.

Out of scope:
- voice calls;
- video calls;
- payments;
- advanced search;
- stories/status.
```

Non-goals help prevent overengineering.

They also tell the interviewer that you are consciously prioritizing.

---

# 8. Stage 2 — Estimate the Scale

After clarifying requirements, estimate the workload if scale affects the design.

Typical quantities include:

- daily active users;
- requests per second;
- peak requests per second;
- read/write ratio;
- storage growth;
- bandwidth;
- concurrent connections.

The goal is not mathematical precision.

The goal is to determine whether the architecture needs certain capabilities.

---

# 9. Example: Messaging Scale Estimate

Suppose the interviewer says:

```text
100 million daily active users
20 messages per user per day
```

Messages per day:

```text
100,000,000 × 20
= 2,000,000,000 messages/day
```

Average messages per second:

```text
2,000,000,000 / 86,400
≈ 23,000 messages/second
```

Peak traffic could be several times higher.

Suppose peak traffic is 5× the average:

```text
≈ 115,000 messages/second
```

That estimate immediately tells us something important.

A single server is not enough.

We probably need:

```text
Load balancer
     ↓
Multiple messaging servers
```

The storage system also needs to support large write volume.

If users maintain persistent connections, we also need to think about the number of simultaneous connections.

Notice the pattern:

```text
Estimate
   ↓
Architectural consequence
```

Do not calculate numbers just to impress the interviewer.

Every meaningful estimate should support a design decision.

---

# 10. Stage 3 — Define APIs or Events

Once you understand the scope, describe how clients interact with the system.

For a simple messaging system:

```http
POST /v1/messages
GET /v1/conversations/{conversation_id}/messages
```

A message creation request might conceptually contain:

```json
{
  "conversation_id": "c123",
  "text": "hello"
}
```

You do not need to specify every field.

Focus on what matters to the design.

Important API considerations include:

- authentication;
- identifiers;
- pagination;
- error handling;
- retry behavior;
- idempotency.

---

# 11. APIs vs Events

APIs are typically used for direct request/response communication.

Example:

```text
Client
  |
  | POST /messages
  v
Messaging Service
```

Events are often used for asynchronous communication.

Example:

```text
Messaging Service
      |
      | MessageCreated
      v
Event Stream
   /       \
  v         v
Analytics  Notification Service
```

Examples of events:

```text
MessageCreated
PaymentCompleted
VideoUploaded
UserRegistered
```

You will study queues, Pub/Sub, and event streams later in the course.

For now, understand this simple distinction:

```text
API
→ request/response interaction

Event
→ something happened and other components may react
```

---

# 12. Stage 4 — Define the Data Model

Before deciding whether to use PostgreSQL, DynamoDB, Cassandra, MongoDB, or anything else, identify the core data.

For messaging, possible entities are:

```text
User
Conversation
ConversationMember
Message
```

A simplified Message model:

```text
Message
-------
message_id
conversation_id
sender_id
content
created_at
```

Now identify the important access patterns.

Example:

```text
1. Fetch recent messages for a conversation.
2. Fetch conversations belonging to a user.
3. Store a newly sent message.
4. Fetch older messages using pagination.
```

These access patterns influence:

- database choice;
- indexes;
- partition keys;
- caching;
- consistency;
- ordering.

The important rule is:

> **Access patterns should influence the data model and database choice.**

Do not start with:

> “I want to use Cassandra.”

Start with:

> “The system needs high write throughput and efficient retrieval of messages by conversation.”

Then choose storage based on that need.

---

# 13. Stage 5 — Build the High-Level Architecture

Once requirements, scale, APIs, and data are reasonably clear, create the simplest architecture that can satisfy them.

Start small.

Example:

```text
Client
  |
  v
Application Server
  |
  v
Database
```

If the workload requires horizontal scaling:

```text
Clients
   |
   v
Load Balancer
   |
   v
+--------------------+
| Application Server |
| Application Server |
| Application Server |
+--------------------+
   |
   v
Database
```

If frequently accessed data creates too much database load:

```text
Clients
   |
   v
Load Balancer
   |
   v
Application Servers
   |
   +------> Cache
   |
   v
Database
```

If expensive work does not need to happen synchronously:

```text
Application Service
        |
        v
      Queue
        |
        v
     Workers
```

Notice that complexity is added only when a requirement justifies it.

This is a key system design habit:

> **Start simple, then evolve.**

---

# 14. Why Starting Simple Matters

Consider two candidates.

Candidate A starts with:

```text
Client
  ↓
API Gateway
  ↓
10 microservices
  ↓
Kafka
  ↓
Redis
  ↓
Cassandra
  ↓
Elasticsearch
  ↓
multiple regions
```

Candidate B starts with:

```text
Client
  ↓
Service
  ↓
Database
```

Then Candidate B explains:

> “At our estimated scale, the database becomes the first major read bottleneck, so I’ll add a cache.”

Then:

> “The application tier needs horizontal scaling, so I’ll add a load balancer and multiple stateless instances.”

Candidate B usually demonstrates stronger engineering reasoning.

The architecture evolves from requirements rather than from memorized diagrams.

---

# 15. Stage 6 — Walk Through the Main Request Flow

A system diagram is not enough.

You need to explain how requests move through the system.

Suppose a user sends a message.

A possible flow:

```text
1. The client sends POST /messages.
2. The load balancer selects a messaging server.
3. The server authenticates the user.
4. The request is validated.
5. The message is stored durably.
6. The receiver is notified or the message is queued for delivery.
7. The sender receives acknowledgement.
```

This walkthrough helps reveal missing pieces.

For example:

```text
What if the database write fails?
What if the receiver is offline?
What if the request is retried?
What if delivery succeeds but the acknowledgement is lost?
```

These questions naturally lead into reliability and correctness.

---

# 16. Read Flow vs Write Flow

In many systems, explicitly explain both.

Example: News Feed.

### Write flow

```text
User creates post
      ↓
Post Service
      ↓
Post Database
      ↓
Feed generation
```

### Read flow

```text
User opens feed
      ↓
Feed Service
      ↓
Feed Cache / Database
      ↓
Posts returned
```

The read and write paths may have very different bottlenecks.

That is why narrating them separately is useful.

---

# 17. Stage 7 — Choose a Deep Dive

You cannot deeply analyze every part of the architecture.

The interviewer usually expects you to go deeper into one or two important challenges.

Examples:

### Messaging

```text
Message delivery
Ordering
Connection management
Offline users
```

### News Feed

```text
Fan-out strategy
Celebrity users
Ranking
Feed caching
```

### Ticket Booking

```text
Seat reservation
Concurrency
Payment failure
Double-booking prevention
```

The deep dive should follow the hardest requirement.

Do not deep-dive into an irrelevant component just because you know it well.

---

# 18. Stage 8 — Scaling the System

Once you have a working architecture, test its limits.

Ask:

```text
What happens if traffic becomes 10× larger?
```

Possible bottlenecks:

- application servers;
- database CPU;
- database storage;
- database connections;
- network bandwidth;
- cache capacity;
- queue backlog;
- a hot partition;
- a single dependency.

Example:

```text
Database handles 10,000 QPS today.

Traffic grows to 100,000 QPS.
```

Possible responses:

```text
Add caching
Add read replicas
Partition the data
Reduce unnecessary reads
Precompute expensive results
```

The correct choice depends on the workload.

---

# 19. Stage 9 — Failure Handling

Distributed systems fail.

A good system design interview should not discuss only the happy path.

Ask questions such as:

```text
What happens if this server crashes?
What happens if the database is unavailable?
What happens if the cache is down?
What happens if a queue consumer stops?
What happens if a request times out?
What happens if the client retries?
What happens if one region becomes unavailable?
```

Failure thinking is one of the biggest differences between a basic architecture diagram and a production-oriented design.

You will learn specific mechanisms later:

- retries;
- exponential backoff;
- circuit breakers;
- idempotency;
- replication;
- failover;
- queues;
- dead-letter queues;
- disaster recovery.

For now, develop the habit:

> Every important component should make you ask, “What if this fails?”

---

# 20. Stage 10 — Discuss Trade-Offs

Almost every architecture decision has a cost.

Good system design is not:

```text
Correct solution vs wrong solution
```

It is often:

```text
Option A
vs
Option B
```

where each option optimizes for different properties.

---

# 21. Example Trade-Off: Strong vs Eventual Consistency

Imagine a social-media like counter.

Strong consistency means users always observe the latest committed count.

Advantages:

```text
More predictable state
```

Possible disadvantages:

```text
Higher coordination cost
Potentially higher latency
Lower availability during some failures
```

Eventual consistency allows replicas or derived state to temporarily disagree.

Advantages:

```text
High availability
Good scalability
Less coordination
```

Disadvantages:

```text
Users may temporarily see stale values
```

For a like count, eventual consistency may be acceptable.

For selling the last available concert ticket, correctness requirements are much stronger.

The right answer depends on the business requirement.

---

# 22. Example Trade-Off: SQL vs NoSQL

Suppose you need:

- transactions;
- relational constraints;
- joins;
- strong integrity.

A relational database may be a natural choice.

Suppose you need:

- enormous horizontal write scale;
- simple key-based access;
- limited relational queries.

A distributed NoSQL design may be more attractive.

Neither is automatically better.

The interviewer wants to hear:

> “Given these access patterns and correctness requirements, I prefer X because Y. The downside is Z.”

---

# 23. Stage 11 — Close the Interview

Near the end, summarize the architecture.

A good closing summary should contain:

1. the main architecture;
2. the most important design decisions;
3. the main trade-offs;
4. the largest remaining risk or bottleneck;
5. how the design could evolve.

Example:

```text
We designed a horizontally scalable messaging system.

Clients connect through a load-balanced application layer.
Messages are persisted before acknowledgement to preserve durability.

We separate delivery from persistence so temporary receiver
unavailability does not lose messages.

The main trade-off is that presence information can be eventually
consistent while message persistence requires stronger guarantees.

At higher scale, message storage and connection management can be
partitioned by region and conversation.
```

This gives the interviewer a clear final picture.

---

# 24. Recommended Interview Time Budget

A useful default for a 45–55 minute interview:

| Time | Activity |
|---:|---|
| 0–6 min | Requirements and scope |
| 6–10 min | Scale estimation |
| 10–16 min | APIs/events and data model |
| 16–28 min | High-level architecture |
| 28–42 min | Deep dive |
| 42–49 min | Scaling, failures, production concerns |
| 49–52 min | Summary |

This is not a strict script.

Sometimes an interviewer may immediately ask you to deep-dive into storage or consistency.

Adapt when needed.

The purpose of the time budget is to prevent common failures such as:

```text
25 minutes clarifying requirements
20 minutes drawing architecture
0 minutes discussing failures
```

or:

```text
20 minutes doing capacity calculations
```

Time management is part of the interview.

---

# 25. The Full Framework in One Example

Prompt:

> Design a URL shortener.

We will study this system properly later. For now, use it only to understand the framework.

---

## Step 1 — Requirements

Functional:

```text
Create a short URL.
Redirect a short URL to the original URL.
```

Non-functional:

```text
Redirects should have low latency.
The system should be highly available.
```

Non-goals:

```text
Advanced analytics
Advertising
Malware detection
```

---

## Step 2 — Scale

Suppose:

```text
10 million new URLs per day
100 million redirects per day
```

Observation:

```text
Reads are much more frequent than writes.
```

That suggests caching may become useful.

---

## Step 3 — API

```http
POST /v1/urls
GET /{short_code}
```

---

## Step 4 — Data Model

```text
ShortURL
--------
short_code
original_url
created_at
expires_at
```

Access patterns:

```text
Create mapping by short_code
Lookup original_url by short_code
```

---

## Step 5 — Initial Architecture

```text
Client
  |
  v
Load Balancer
  |
  v
URL Service
  |
  v
Database
```

---

## Step 6 — Request Flow

Redirect:

```text
GET /abc123
   ↓
URL Service
   ↓
Database lookup
   ↓
301/302 redirect
```

---

## Step 7 — Deep Dive

Possible deep dive:

```text
How are unique short codes generated?
```

---

## Step 8 — Scale

If redirects grow dramatically:

```text
Add cache
Add application servers
Possibly partition mappings
```

---

## Step 9 — Failures

Ask:

```text
What if the cache fails?
What if the database leader fails?
What if two requests generate the same alias?
```

---

## Step 10 — Trade-Offs

Possible trade-off:

```text
Random IDs
vs
sequence-based IDs
```

---

# 26. Common Beginner Mistakes

## Mistake 1 — Immediately Naming Technologies

Bad:

```text
Let's use Kafka, Redis, and Cassandra.
```

Better:

```text
This operation can be asynchronous, so I want a durable
queue between the API and workers.
```

Then you may mention Kafka, SQS, RabbitMQ, etc. as implementations if relevant.

---

## Mistake 2 — Designing Before Understanding Requirements

Bad:

```text
Interviewer: Design Dropbox.

Candidate immediately draws architecture.
```

Better:

```text
Should we focus on file upload/download and synchronization,
or also collaborative editing?
```

---

## Mistake 3 — Doing Too Much Math

Estimation exists to support decisions.

Do not spend ten minutes computing exact storage to three decimal places.

Approximation is enough.

---

## Mistake 4 — Choosing a Database Before Understanding Access Patterns

Bad:

```text
We'll use MongoDB.
```

Better:

```text
The main access pattern is lookup by user ID with very few joins.
Let's first evaluate whether a key-value/document model fits.
```

---

## Mistake 5 — Overengineering

Do not automatically add:

- microservices;
- Kafka;
- Redis;
- multiple regions;
- sharding;
- Kubernetes;
- CQRS;
- event sourcing.

Every component should solve a real requirement.

---

## Mistake 6 — Ignoring the Request Flow

A diagram with boxes and arrows can still be incomplete.

Explain what happens when the user performs important operations.

---

## Mistake 7 — Ignoring Failure

A production system must handle:

```text
slow components
crashes
timeouts
duplicates
partial failure
traffic spikes
```

At minimum, acknowledge these.

---

## Mistake 8 — Trying to Cover Everything

A focused design is better than superficial coverage of every possible feature.

Prioritize.

---

# 27. A Mental Model for Every Architecture Component

Whenever you draw a component, be prepared to answer five questions.

Example component:

```text
Cache
```

Ask:

```text
1. Why is it here?
2. What data does it contain?
3. What problem does it solve?
4. What happens if it fails?
5. What is the trade-off?
```

Example answer:

```text
The cache stores frequently requested product data.

It reduces database reads and improves latency.

The database remains the source of truth.

If the cache fails, requests can temporarily fall back to the database,
but we need load protection.

The trade-off is stale data and cache invalidation complexity.
```

That style of explanation is much stronger than simply drawing a Redis box.

---

# 28. A Mental Model for Every Design Decision

Use this template:

```text
Requirement
    ↓
Observation
    ↓
Decision
    ↓
Benefit
    ↓
Trade-off
```

Example:

```text
Requirement:
Feed reads need low latency.

Observation:
The same feed data is requested repeatedly.

Decision:
Cache prepared feed entries.

Benefit:
Lower read latency and database load.

Trade-off:
Cached feeds can become stale and require invalidation/update logic.
```

This is one of the most reusable habits in system design.

---

# 29. What L4 Depth Usually Looks Like

For L4, your explanation should usually reach the level of:

```text
What component do I need?
Why do I need it?
How does it work conceptually?
What trade-off am I accepting?
What happens when it fails?
```

You usually do **not** need to implement advanced distributed algorithms from first principles.

For example, if discussing replication:

Good L4-level explanation:

```text
I would use leader/follower replication so reads can scale and
the system can tolerate a replica failure.

Because replication is asynchronous, users reading from a replica
may briefly observe stale data.

For operations that require read-after-write consistency, I may route
the immediate read to the leader.
```

You usually do not need to derive a consensus algorithm.

The course will gradually build this depth across the fundamentals.

---

# 30. What You Should Remember From This Lesson

The most important ideas are:

### 1. Requirements come before architecture.

Do not solve the wrong problem.

### 2. Estimate only what influences decisions.

Numbers are useful when they change architecture.

### 3. APIs and data models describe the system's contracts and state.

Think about access patterns before choosing a database.

### 4. Start with a simple architecture.

Add complexity only when justified.

### 5. Explain request flows.

Show that the design works end to end.

### 6. Deep-dive into the hardest requirement.

Do not try to deeply analyze everything.

### 7. Think about failures.

Distributed systems fail.

### 8. Discuss trade-offs.

There is rarely one perfect solution.

### 9. Communicate your reasoning.

The interviewer is evaluating how you think, not only the final diagram.

---

# 31. One-Page Study Summary

```text
SYSTEM DESIGN INTERVIEW FRAMEWORK

1. REQUIREMENTS
   Functional
   Non-functional
   Non-goals

2. SCALE
   DAU
   QPS
   Peak traffic
   Storage
   Bandwidth
   Concurrency
   Skew

3. CONTRACTS
   APIs
   Events
   Retry behavior

4. DATA
   Entities
   Access patterns
   Source of truth
   Consistency

5. HIGH-LEVEL DESIGN
   Start simple
   Add components only for requirements

6. REQUEST FLOWS
   Main write flow
   Main read flow

7. DEEP DIVE
   Focus on hardest requirement

8. SCALE
   What breaks at 10×?

9. FAILURE
   What if this component is slow/down?
   What about retries and duplicates?

10. TRADE-OFFS
    Alternatives
    Why this choice?

11. SUMMARY
    Architecture
    Decisions
    Trade-offs
    Limitation
    Evolution
```

---

# 32. Review Questions

These questions are for checking understanding **after reading the lesson**.

You do not need to answer them while studying the documentation.

1. Why should requirements be clarified before architecture?
2. What is the difference between functional and non-functional requirements?
3. Why do system design interviews use capacity estimates?
4. Why should access patterns come before database selection?
5. Why should an initial architecture be simple?
6. What is the purpose of narrating request flows?
7. How should you choose a deep-dive topic?
8. What kinds of failure questions should you ask?
9. What does “trade-off” mean?
10. What should be included in the closing summary?

---

# 33. Lesson Completion Check

You are ready for Lesson 2 when you can explain, without reading:

```text
Requirements
Scale
APIs/events
Data model
High-level architecture
Request flows
Deep dive
Scaling
Failures
Trade-offs
Summary
```

You should also be able to explain this principle:

> **Requirement → capability → design choice → trade-off**

If that sequence makes sense, Lesson 1 has done its job.

---

# 34. Personal Notes

Use this section as your study documentation.

## Concepts that are clear

```text

```

## Concepts to revisit

```text

```

## Important points I want to remember

```text
1.
2.
3.
```

## Questions for later

```text

```

---

# End of Lesson 1

**Next:** Lesson 2 — Back-of-the-Envelope Estimation
