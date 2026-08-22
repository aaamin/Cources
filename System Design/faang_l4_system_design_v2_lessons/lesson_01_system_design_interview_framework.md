# Lesson 1 — System Design Interview Framework

**Course:** FAANG L4 System Design Interview Course Plan — V2  
**Session:** 1 / 46  
**Recommended time:** 60–90 minutes  
**Phase:** Fundamentals

---

# 1. Lesson Goal

By the end of this session, you should understand the complete structure of a system design interview and be able to explain **what you should do at each stage**.

You are **not** expected to design a complex system in this lesson.

The goal is to learn the reusable interview framework:

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

This framework will be reused throughout the course.

---

# 2. What a System Design Interview Is Testing

A system design interview is not primarily testing whether you know the exact architecture of a famous product.

It is testing whether you can:

- turn an ambiguous problem into clear requirements;
- identify the important constraints;
- estimate scale when needed;
- define sensible APIs and data models;
- construct a simple end-to-end architecture;
- identify bottlenecks;
- reason about failures;
- discuss trade-offs;
- communicate clearly while designing.

For L4, the goal is usually not to invent a perfect globally distributed architecture.

A strong answer is more often:

> A simple, reasonable architecture that satisfies the requirements and whose trade-offs you can explain.

---

# 3. The Core Interview Mindset

A weak approach is:

```text
Prompt
  ↓
Immediately draw:
Load Balancer
Redis
Kafka
Cassandra
Kubernetes
CDN
Microservices
```

This is technology-first thinking.

A stronger approach is:

```text
Prompt
  ↓
Understand requirements
  ↓
Understand workload
  ↓
Determine required capabilities
  ↓
Choose architecture
  ↓
Choose technologies only when needed
```

The architecture should follow from the problem.

Do not add components merely because they are common in system design diagrams.

---

# 4. Stage 1 — Clarify Requirements

Recommended interview time:

**0–6 minutes**

The prompt will usually be intentionally incomplete.

Example:

> Design a messaging system.

That leaves many unanswered questions.

You need to clarify the scope before designing.

---

## 4.1 Functional Requirements

Functional requirements describe what users can do.

For a messaging system:

```text
Users can send one-to-one messages.
Users can receive messages.
Users can view message history.
```

Possible additional features:

```text
Group chat
Media attachments
Message editing
Read receipts
Presence
Search
```

You should not automatically support everything.

Prioritize the core features.

---

## 4.2 Non-Functional Requirements

Non-functional requirements describe how the system should behave.

Examples:

- low latency;
- high availability;
- durability;
- consistency;
- scalability;
- reliability;
- security;
- cost efficiency.

For a messaging system:

```text
Messages should normally be delivered quickly.
Messages should not be lost.
The system should support many concurrent connections.
```

---

## 4.3 Define Non-Goals

Explicitly state what is not being designed.

Example:

```text
For this interview, I will focus on:
- one-to-one text messaging;
- message history;
- online/offline delivery.

I will leave:
- voice/video calling;
- payments;
- advanced search;
- stories/status

out of scope.
```

This prevents the design from becoming unnecessarily large.

---

# 5. Stage 2 — Estimate Scale

Recommended interview time:

**6–10 minutes**

System design is heavily affected by workload.

You may need to estimate:

- daily active users;
- requests per second;
- peak QPS;
- storage;
- bandwidth;
- concurrent connections;
- read/write ratio.

But estimates are useful only if they influence architecture.

---

## Example

Suppose:

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
≈ 23,000 messages/sec
```

Peak traffic might be several times higher.

This tells us:

- one application server cannot handle everything;
- the storage layer must scale;
- message processing may need partitioning;
- persistent connections may be important.

The exact number is less important than the architectural consequence.

---

# 6. Stage 3 — Define APIs and Events

Recommended interview time:

**10–16 minutes**, together with data modeling.

APIs define how clients interact with your system.

For a messaging system:

```http
POST /v1/messages
GET /v1/conversations/{conversation_id}/messages
```

Possible request:

```json
{
  "conversation_id": "c123",
  "sender_id": "u10",
  "text": "hello"
}
```

You should think about:

- request fields;
- response fields;
- authentication;
- pagination;
- errors;
- retries;
- idempotency.

---

## Events

Some systems also communicate asynchronously.

Example:

```text
MessageCreated
NotificationRequested
VideoUploaded
PaymentCompleted
```

Events become important later when learning queues, Pub/Sub, and streaming.

For now, remember:

> APIs are often synchronous contracts. Events are often asynchronous contracts.

---

# 7. Stage 4 — Define the Data Model

Before selecting a database product, identify the main entities and access patterns.

For messaging:

```text
User
Conversation
ConversationMember
Message
```

A simplified Message entity:

```text
Message
-------
message_id
conversation_id
sender_id
content
created_at
```

---

## Access Patterns Matter

Ask:

```text
How do we find a message?
How do we fetch the latest messages in a conversation?
How do we fetch conversations for a user?
Do messages require ordering?
```

These questions influence:

- indexes;
- partition keys;
- SQL vs NoSQL;
- caching;
- consistency requirements.

A critical principle:

> Choose storage based on access patterns and correctness requirements.

Not:

> Choose MongoDB because it scales.

---

# 8. Stage 5 — Build a High-Level Architecture

Recommended interview time:

**16–28 minutes**

Start simple.

Example:

```text
Clients
   |
   v
Load Balancer
   |
   v
Application Servers
   |
   v
Database
```

That may be enough for the first version.

Only add complexity when requirements demand it.

As the system grows:

```text
Clients
   |
   v
Load Balancer
   |
   v
Application Services
   |
   +----> Cache
   |
   +----> Database
   |
   +----> Queue
```

The important point is not the number of boxes.

The important point is:

> Can you explain what each box does and why it exists?

---

# 9. Stage 6 — Walk Through the Main Request Flow

A diagram without a request flow is incomplete.

Suppose a user sends a message.

Walk through it:

```text
1. Client sends POST /messages.
2. Request reaches the load balancer.
3. Load balancer routes it to a messaging server.
4. Server validates the request.
5. Message is written to storage.
6. Delivery process sends it to the receiver.
7. Server returns acknowledgement to sender.
```

If the receiver is offline:

```text
Message remains durable in storage.
Receiver fetches or receives it when reconnecting.
```

Narrating the flow proves that the architecture works end-to-end.

---

# 10. Stage 7 — Deep Dive

Recommended interview time:

**28–42 minutes**

You cannot deeply analyze every component.

Choose the most important challenge.

For chat:

```text
Connection management
Message delivery
Ordering
Offline users
```

For a news feed:

```text
Fan-out
Celebrity traffic
Ranking
Caching
```

For ticket booking:

```text
Seat reservation
Concurrency
Payment timeout
Double booking
```

The deep dive should be driven by the hardest requirement.

---

# 11. Stage 8 — Scaling and Failure Handling

After the happy path works, challenge the architecture.

Ask:

```text
What happens at 10× traffic?
What fails first?
What happens if the cache goes down?
What happens if a database replica is unavailable?
What happens if one partition gets most of the traffic?
What happens if a request is retried?
```

These questions reveal where your design is weak.

You will learn specific failure-handling patterns in later sessions.

For now, develop the habit of asking:

> What happens when this component fails?

---

# 12. Stage 9 — Discuss Trade-Offs

There is rarely one perfect solution.

Good system design is about trade-offs.

Example:

### Option A — Strong consistency

Benefit:

```text
Users always see the latest committed state.
```

Cost:

```text
Potentially higher latency and lower availability during failures.
```

### Option B — Eventual consistency

Benefit:

```text
Higher availability and easier scaling for some workloads.
```

Cost:

```text
Users may temporarily see stale data.
```

A strong candidate can explain both choices and justify one.

---

# 13. Stage 10 — Closing Summary

Reserve approximately **2–3 minutes**.

A concise closing summary should answer:

1. What architecture did you choose?
2. What were the most important decisions?
3. What trade-offs did you make?
4. What is the largest remaining limitation?
5. How could the system evolve later?

Example:

```text
We designed a horizontally scalable messaging service with
persistent client connections, durable message storage, and
asynchronous delivery.

Messages are partitioned by conversation to preserve local
ordering and distribute load.

The main trade-off is that some presence information may be
eventually consistent.

At much larger scale, we would further partition connection
management and message storage by region.
```

---

# 14. Recommended Interview Time Budget

A useful default:

| Time | Activity |
|---:|---|
| 0–6 min | Requirements and scope |
| 6–10 min | Estimation |
| 10–16 min | APIs / events / data model |
| 16–28 min | High-level architecture and flows |
| 28–42 min | Deep dive |
| 42–49 min | Failures, scaling, production concerns |
| 49–52 min | Summary |

This is a guide, not a rigid script.

If the interviewer redirects the discussion, adapt.

---

# 15. Common Beginner Mistakes

## Mistake 1 — Designing Before Clarifying

Bad:

```text
Prompt: Design Twitter.

Candidate:
We'll use Kafka, Redis, Cassandra...
```

Better:

```text
First I'd like to clarify the primary features we're designing.
Should we focus on posting and timeline generation?
```

---

## Mistake 2 — Using Technology Names Without Reasons

Bad:

```text
We'll use Kafka.
```

Better:

```text
Feed-generation work does not have to happen synchronously,
so I would introduce a durable queue or event stream here.
Kafka could be one implementation if replay and high-throughput
stream consumption are useful.
```

Capabilities first, product second.

---

## Mistake 3 — Overengineering Immediately

Bad first architecture:

```text
20 microservices
Kafka
Redis
Cassandra
ElasticSearch
ZooKeeper
Kubernetes
multiple regions
```

Better:

```text
Client
  ↓
Service
  ↓
Database
```

Then scale where needed.

---

## Mistake 4 — Only Drawing Boxes

You must explain:

```text
What happens when a user writes?
What happens when a user reads?
```

---

## Mistake 5 — Ignoring Failures

Happy paths are not enough.

Ask:

```text
What if this server dies?
What if this request is duplicated?
What if storage becomes slow?
```

---

## Mistake 6 — Trying to Design Everything

Scope aggressively.

Interview time is limited.

It is better to deeply design the important 20% than superficially mention 100 features.

---

# 16. Example Mini Walkthrough

Prompt:

> Design a URL shortener.

You are **not solving this fully today**.

The purpose is to see the framework.

---

## Requirements

```text
Create short URLs.
Redirect short URL to original URL.
URLs should have low redirect latency.
```

Non-goals:

```text
Advanced analytics
Advertisement platform
Malware detection
```

---

## Scale

Suppose:

```text
10 million new URLs/day
100 million redirects/day
```

Observation:

```text
The workload is read-heavy.
Caching may later be useful.
```

---

## APIs

```http
POST /v1/urls
GET /{short_code}
```

---

## Data

```text
ShortURL
--------
short_code
original_url
created_at
expires_at
```

---

## Initial Architecture

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

## Possible Deep Dive

```text
How are short IDs generated?
```

---

## Scaling Question

```text
What if redirects increase 100×?
```

Possible direction:

```text
Introduce caching and additional application servers.
```

Again, the goal today is not the final URL-shortener architecture.

The goal is to recognize the sequence.

---

# 17. Small Application Drills

Do these without trying to build full architectures.

---

## Drill 1 — Identify Requirements

Prompt:

> Design a notification system.

Write:

### Three functional requirements

1.
2.
3.

### Three non-functional requirements

1.
2.
3.

### Three non-goals

1.
2.
3.

---

## Drill 2 — Identify the Hard Part

For each system, choose one likely deep-dive area.

### WhatsApp

```text
Your answer:
```

### YouTube

```text
Your answer:
```

### Ticketmaster

```text
Your answer:
```

### News Feed

```text
Your answer:
```

---

## Drill 3 — Architecture Before Technology

For each problem, describe the capability you need before naming a product.

Example:

Bad:

```text
Need Redis.
```

Better:

```text
Need a low-latency cache for frequently accessed data.
```

Now complete:

### Notifications

```text
Need:
```

### Video upload

```text
Need:
```

### Chat

```text
Need:
```

### Search

```text
Need:
```

---

# 18. Spoken Exercise

Without looking at the framework, explain aloud:

> “How would I approach an unfamiliar system design interview question?”

Your explanation should approximately cover:

```text
1. Clarify requirements.
2. Identify non-functional priorities.
3. Estimate scale.
4. Define APIs/events.
5. Define data and access patterns.
6. Draw a simple architecture.
7. Walk through requests.
8. Deep dive into the hardest requirement.
9. Discuss bottlenecks and failures.
10. Discuss trade-offs and summarize.
```

Try to explain this in **3–5 minutes**.

---

# 19. Knowledge Check

Answer without looking back.

### Question 1

Why should requirements be clarified before drawing architecture?

### Question 2

What is the difference between a functional and a non-functional requirement?

### Question 3

Why should you estimate scale?

### Question 4

What should come first: database product selection or access-pattern analysis?

### Question 5

Why should the first architecture usually be simple?

### Question 6

What is a request-flow walkthrough?

### Question 7

Why can't you deep-dive every component?

### Question 8

Give three failure questions you should ask about an architecture.

### Question 9

What does “trade-off” mean in system design?

### Question 10

What should your closing summary contain?

---

# 20. Knowledge Check — Suggested Answers

Do not read these until you attempt the questions.

---

### Answer 1

Because the architecture should solve the actual required problem. Designing before clarifying scope often causes unnecessary complexity or misses important requirements.

### Answer 2

A functional requirement describes what the system does. A non-functional requirement describes qualities or constraints such as latency, availability, consistency, durability, or scale.

### Answer 3

Scale can change architectural decisions. Estimates help identify whether the system requires horizontal scaling, partitioning, caching, high bandwidth, large storage, or large numbers of persistent connections.

### Answer 4

Access-pattern analysis should come first. The database should be selected based on queries, relationships, correctness requirements, scale, and workload.

### Answer 5

A simple architecture is easier to reason about and may already satisfy the requirements. Complexity should be added only when a specific requirement or bottleneck justifies it.

### Answer 6

It is a step-by-step explanation of what happens through the architecture when a user performs an important operation such as a read or write.

### Answer 7

Interview time is limited. Deep dives should focus on the most difficult or important requirements.

### Answer 8

Examples:

- What happens if the database is unavailable?
- What happens if a request is retried?
- What happens if traffic becomes 10× higher?
- What happens if the cache fails?
- What happens if one partition becomes hot?

### Answer 9

A trade-off means gaining one property while accepting a cost or weakness elsewhere—for example, stronger consistency may increase latency or reduce availability during some failures.

### Answer 10

The main architecture, important design decisions, key trade-offs, major limitation, and possible evolution path.

---

# 21. Lesson Completion Checklist

Mark Session 1 complete only when you can do all of the following:

- [ ] Explain the overall system design interview framework without notes.
- [ ] Distinguish functional and non-functional requirements.
- [ ] Explain why scope and non-goals matter.
- [ ] Explain why scale estimation is useful.
- [ ] Explain why APIs and data modeling come before detailed infrastructure.
- [ ] Explain why the initial architecture should be simple.
- [ ] Walk through a read or write flow conceptually.
- [ ] Explain what a deep dive is.
- [ ] Give at least three failure questions.
- [ ] Explain what a system design trade-off is.
- [ ] Give a 2-minute closing summary structure.
- [ ] Complete the knowledge check with at least 8/10 correct.
- [ ] Explain the whole interview approach aloud in 3–5 minutes.

---

# 22. Session Notes Template

Use this after completing the lesson.

## Concepts I understand well

```text

```

## Concepts still unclear

```text

```

## Three things I want to remember

1.
2.
3.

## Questions to revisit later

```text

```

---

# 23. One-Page Recall Sheet

Before Session 2, try to reproduce this from memory:

```text
SYSTEM DESIGN INTERVIEW

1. REQUIREMENTS
   - Functional
   - Non-functional
   - Non-goals

2. SCALE
   - QPS
   - storage
   - bandwidth
   - concurrency
   - peak/skew

3. CONTRACTS
   - APIs
   - events

4. DATA
   - entities
   - access patterns
   - consistency

5. HIGH-LEVEL DESIGN
   - simple end-to-end architecture

6. FLOWS
   - main read
   - main write

7. DEEP DIVE
   - hardest requirement

8. SCALE + FAILURE
   - bottlenecks
   - retries
   - failures
   - 10× traffic

9. TRADE-OFFS
   - alternatives
   - why this choice?

10. SUMMARY
   - architecture
   - decisions
   - limitations
   - evolution
```

---

# End of Lesson 1

**Next session:** Lesson 2 — Back-of-the-Envelope Estimation
