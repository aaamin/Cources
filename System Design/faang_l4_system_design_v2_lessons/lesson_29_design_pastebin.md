# Lesson 29 — Design Pastebin

**Course:** FAANG L4 System Design Interview Course Plan — V2  
**Session:** 29 / 46  
**Phase:** Guided Design  
**Recommended time:** 90–120 minutes

---

## 1. Objective

This is a **full design practice session**.

Attempt the problem before studying external solutions. The goal is to derive the architecture from requirements using the V2 interview framework.

### Main concepts to exercise

- create/read APIs
- ID generation
- payload storage
- expiration
- large pastes
- read-heavy access
- caching
- abuse constraints
- storage growth
- simple-to-scaled evolution

---

## 2. Interview Prompt

> **Design Pastebin**

Treat this as an interviewer prompt, not a request to reproduce a known architecture.

Before drawing anything, clarify the scope.

---

## 3. Closed-Book Attempt

Use approximately 45–55 minutes.

### 0–6 min — Requirements

Clarify:

- core user actions;
- non-goals;
- latency expectations;
- availability/durability;
- consistency/correctness;
- geography;
- retention;
- security/abuse constraints when relevant.

### 6–10 min — Scale

Estimate only numbers that change decisions:

```text
DAU:
Average QPS:
Peak QPS:
Read/write ratio:
Payload size:
Storage growth:
Bandwidth:
Concurrent connections:
Skew/hot-key risk:
```

### 10–16 min — APIs / Events / Data Model

Define:

- important APIs/events;
- retry/idempotency behavior;
- core entities;
- source of truth;
- important indexes;
- partition candidates;
- retention.

### 16–28 min — High-Level Design

Start simple and complete.

```text
Clients
   |
   v
Edge / Load Balancer / Gateway
   |
   v
Core Service(s)
   |
   +----> Cache
   +----> Database
   +----> Queue / Stream
   +----> Object Storage
```

Only include components justified by the workload.

### 28–42 min — Deep Dive

Pick the hardest requirement and go deep.

### 42–49 min — Production Review

Discuss:

- bottlenecks;
- hot spots/skew;
- retries and duplicates;
- partial failures;
- backpressure;
- consistency;
- security;
- observability;
- multi-region/cost where relevant.

### 49–52 min — Summary

State:

1. the main design;
2. the most important trade-off;
3. the current bottleneck;
4. the next evolution at 10× scale.

---

## 4. Required Change Request

After your initial architecture is coherent, introduce this change:

> **Support very large pastes and configurable expiration.**

Do not restart the design. Adapt it.

This is important: real interviews often change requirements midstream.

---

## 5. Deep-Dive Guide

Use these prompts only **after** your first independent attempt.

### 1. Create/Read Apis

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 2. Id Generation

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 3. Payload Storage

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 4. Expiration

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 5. Large Pastes

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 6. Read-Heavy Access

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 7. Caching

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 8. Abuse Constraints

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 9. Storage Growth

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?

### 10. Simple-To-Scaled Evolution

Ask:

- Why is this relevant to the system?
- What is the simplest workable approach?
- What breaks under skew or failure?
- What state must be durable?
- What consistency or ordering is required?
- How would you observe whether it is working?


---

## 6. Failure Injection

Inject at least three of these:

- cache unavailable;
- primary database unavailable;
- slow dependency;
- duplicate request/event;
- one shard or tenant becomes hot;
- network partition;
- worker crashes after partial work;
- regional outage;
- downstream provider rate-limits you.

For each:

```text
Failure:
User impact:
Detection:
Immediate behavior:
Recovery:
Longer-term improvement:
```

---

## 7. 40-Point Scoring Rubric

| Category | 0 | 2 — Developing | 4 — Strong |
|---|---|---|---|
| Requirements & Scope | Does not clarify | Finds main requirements but misses priorities | Controls ambiguity, priorities, constraints, and non-goals |
| Estimation & Workload | Missing/irrelevant | Reasonable numbers, weak linkage | Peak/skew/growth tied to design decisions |
| APIs / Events / Data Model | Missing/incoherent | Happy path and main entities | Access-pattern driven; errors, idempotency, ownership, indexes |
| High-Level Design & Flows | Incomplete | Coherent happy path | Simple complete design with clear read/write/async flows |
| Scalability & Performance | No bottleneck reasoning | Common scaling components | Workload-specific limits, hot spots, backpressure, 10× path |
| Correctness & Consistency | Ignored/contradictory | Names a model | Invariants, races, ordering, duplicates, recovery are precise |
| Reliability & Operations | Ignores failures | Basic redundancy/monitoring | Failure domains, degradation, observability, deploy/restore |
| Security / Privacy / Cost | Ignored | Mentions basics | Prioritizes meaningful controls and dominant cost drivers |
| Trade-Offs & Evolution | Technology list only | Gives one alternative | Compares credible alternatives and evolves only when justified |
| Communication & Time Control | Hard to follow | Understandable but reactive | Leads clearly, adapts, manages time, summarizes |


### Score interpretation

- **34–40:** strong L4 signal for this prompt.
- **32–33:** passing readiness signal.
- **28–31:** close, but important interview risk remains.
- **24–27:** foundations exist; integration needs repair.
- **Below 24:** return to relevant fundamentals.

A **0 or 1** in requirements, APIs/data, HLD, correctness, trade-offs, or communication is a non-pass.

---

## 8. Review Procedure

After scoring:

1. Identify the bottom two categories.
2. Write the three highest-impact omissions.
3. Review references only now.
4. Redo the weakest 15–20 minutes from a blank page.
5. Explain the revised design in 5 minutes.
6. Record one pattern you can reuse elsewhere.

### Repair log

```text
Original score:

Weak category #1:
Repair:

Weak category #2:
Repair:

Three biggest misses:
1.
2.
3.

Reusable pattern:
```

---

## 9. Completion Checklist

- [ ] I attempted the design before reading a solution.
- [ ] I clarified requirements and non-goals.
- [ ] My estimates influenced at least one decision.
- [ ] I defined APIs/events and a data model.
- [ ] I built a complete end-to-end architecture.
- [ ] I narrated a main read/write flow.
- [ ] I completed a requirement-driven deep dive.
- [ ] I handled the change request without restarting.
- [ ] I injected at least three failures.
- [ ] I discussed at least one credible alternative.
- [ ] I scored the design using the 40-point rubric.
- [ ] I repaired the two weakest areas.

---

## 10. Recall Card

```text
DESIGN PASTEBIN

REQUIREMENTS
SCALE
API / EVENTS
DATA MODEL
HLD
MAIN FLOWS
DEEP DIVE
FAILURES
TRADE-OFFS
CHANGE REQUEST
SUMMARY

KEY LESSONS:
- create/read APIs
- ID generation
- payload storage
- expiration
- large pastes
- read-heavy access
```

---

**Next:** Lesson 30 — Design a Notification Service
