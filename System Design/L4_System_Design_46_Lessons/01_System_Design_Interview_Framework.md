# Session 1 — System Design Interview Framework

## Outcome

By the end of this lesson, you should be able to lead a 45–55 minute L4 system-design interview without losing structure. You should know what to clarify, which estimates matter, how to move from requirements to APIs and data, when to draw the high-level architecture, how to choose a deep dive, and how to close with trade-offs.

## Why This Matters

Many candidates know individual technologies but perform poorly because the interview becomes a technology dump. The interviewer is evaluating whether you can take an ambiguous problem, turn it into explicit requirements, make reasonable assumptions, design a coherent system, identify the hard part, and defend choices.

A strong interview is a sequence of decisions:

```text
Ambiguous problem
    ↓
Clarified requirements
    ↓
Workload and constraints
    ↓
Interfaces and data
    ↓
Simple architecture
    ↓
Hardest requirement
    ↓
Scaling + failures
    ↓
Trade-offs + evolution
```

The structure protects you from spending 25 minutes on one database choice while ignoring the actual system.

## Mental Model

Think in four layers:

1. **What are we building?** — requirements and non-goals.
2. **How much must it handle?** — scale, skew, latency, durability, consistency.
3. **What is the simplest architecture that works?** — APIs, data, services, storage, cache, async paths.
4. **Where is the real difficulty?** — contention, fan-out, ordering, hot keys, geo, realtime, correctness, failures.

## 1. Clarify Requirements

Start with functional requirements. Do not design until you know the core user actions.

For a chat system, examples might be:

- send a one-to-one message;
- receive messages in near real time;
- retrieve conversation history;
- support offline users.

Then identify non-functional priorities. Ask which of these matter most:

- latency;
- availability;
- durability;
- consistency;
- throughput;
- freshness;
- cost;
- geographic scope.

Do not ask twenty questions. Ask only questions that change the architecture.

### Good scope control

If asked to design YouTube, you might say:

> I’ll focus on upload, processing, playback, and metadata. I’ll exclude recommendations, comments, advertising, and creator monetization unless you want one of those included.

This reduces ambiguity and shows control.

## 2. Estimate Only What Can Change the Design

Typical useful numbers:

- DAU/MAU;
- average and peak requests per second;
- read/write ratio;
- object/message size;
- storage growth;
- concurrent persistent connections;
- bandwidth;
- traffic skew.

Bad estimation is arithmetic for its own sake. Good estimation creates a design implication.

Example:

```text
10M daily users
20 reads/user/day
≈ 200M reads/day
≈ 2.3k average RPS
Assume 5× peak → ~12k peak RPS
```

The exact number is not sacred. The important next sentence is:

> This is beyond a single application instance but still manageable with a horizontally scaled stateless service; the read-heavy ratio also makes caching valuable.

## 3. Define APIs / Events

Choose the minimum interfaces needed for the core flows.

Example:

```http
POST /messages
GET /conversations/{id}/messages?cursor=...
```

For async behavior, define important events:

```text
MessageAccepted
NotificationRequested
VideoUploaded
TranscodeCompleted
```

Do not spend too long on endpoint syntax. Focus on semantics:

- What identifies the resource?
- Is the request idempotent?
- How is pagination done?
- What errors matter?
- What happens on retry?

## 4. Model the Important Data

The data model should follow access patterns.

For messaging:

```text
Conversation
Message
Participant
User
```

Ask:

- How are messages fetched?
- What is the ordering key?
- What must be unique?
- What must be durable?
- What needs strong consistency?
- What can be eventually consistent?

Do not choose DynamoDB, Cassandra, PostgreSQL, or MongoDB before understanding these questions.

## 5. Draw a Simple High-Level Architecture

Start simple enough that the entire happy path fits on one screen.

Example:

```text
Client
  |
API Gateway / LB
  |
App Service
  |------ Cache
  |------ Database
  |
 Queue
  |
Workers
```

Then narrate one important read and one important write. This is more useful than drawing fifteen boxes with no explanation.

## 6. Choose the Deep Dive

Spend the largest block of time on the hardest requirement.

Examples:

- Chat → persistent connections, ordering, offline delivery.
- Feed → fan-out strategy and celebrity problem.
- Ticketing → seat hold correctness and payment failure.
- Metrics → partitioning, high write throughput, backpressure.
- Drive → upload synchronization and conflicts.

The deep dive should not be random. It should correspond to the design's biggest risk.

## 7. Discuss Failure and Scaling

For major components, ask:

```text
What if it becomes slow?
What if it fails?
What if traffic becomes 10×?
What if one tenant/key is much hotter than others?
What if the request is retried?
```

Strong interview answers discuss degraded behavior, not just redundancy.

Example:

> If the notification provider is slow, I would not keep user requests waiting. Notifications are queued. The worker uses timeouts and bounded retries with jitter, then sends persistent failures to a DLQ.

## 8. Make Trade-Offs Explicit

A design choice is incomplete until you explain the cost.

Examples:

- fan-out on write gives fast reads but expensive writes;
- asynchronous replication improves write latency but permits stale reads;
- microservices permit independent scaling but add distributed-system complexity;
- caching reduces latency but creates invalidation and stale-data problems.

Use language such as:

> I prefer X because requirement Y matters more than Z. If that priority changed, alternative A would become reasonable.

## 9. Close the Interview

Use the final 2–3 minutes to summarize:

1. core architecture;
2. hardest design decision;
3. important trade-off;
4. major failure-handling strategy;
5. 10× evolution.

A clear close can repair an interview that became slightly messy.

## Suggested Time Budget

For a ~50 minute interview:

| Time | Activity |
|---|---|
| 0–6 min | Requirements and scope |
| 6–10 min | Useful estimates |
| 10–16 min | API/events/data model |
| 16–28 min | High-level architecture and flows |
| 28–42 min | Deep dive |
| 42–49 min | Failures, scaling, security/ops as relevant |
| 49–52 min | Summary |

Do not treat this as a rigid stopwatch. It is a guardrail.

## Worked Mini Example — Design a Simple Reminder Service

### Requirements
- User creates a reminder for a future time.
- Service triggers a notification around that time.
- Duplicate notifications should be minimized.
- Missing a reminder is worse than delivering a duplicate.

### Scale assumption
Suppose 1M reminders/day with bursty morning/evening traffic.

### API
```http
POST /reminders
GET /reminders/{id}
DELETE /reminders/{id}
```

### Data
```text
Reminder(id, user_id, trigger_at, payload, status)
```

Index on `trigger_at` or partition jobs into time buckets.

### Architecture
```text
Client → API → Reminder DB
                  |
               Scheduler
                  |
                Queue
                  |
               Workers → Push/Email provider
```

### Hard part
Scheduling and retries. The system should tolerate scheduler/worker failure.

### Correctness choice
Use at-least-once delivery with idempotent notification processing rather than pretending end-to-end exactly-once is free.

### Failure
If the provider is down, keep the job durable, retry with backoff, and alert when the backlog grows.

The point is not that this architecture is the only solution. The point is that every major choice follows from a requirement.

## Small Design Drills

Try these aloud before reading the answer key.

1. You are asked to design Instagram. What are three scope questions that materially affect architecture?
2. A design has 100 average RPS and a 50× traffic spike every Friday. Which number matters more for capacity?
3. When should you discuss security in the interview?
4. Why is “we use Kafka because it scales” a weak answer?
5. What should determine the deep-dive section?
6. An interviewer changes the requirement halfway through. Should you restart the design?

<details>
<summary>Answer key</summary>

1. Examples: focus on photo upload/feed only? videos? private accounts? required feed freshness? geographic scope?
2. Peak/burst capacity matters for survival, while average still matters for cost.
3. When relevant to the system and again in production-readiness review; do not spend half the interview on generic security controls.
4. It names a technology without identifying the workload, requirement, semantics, or trade-off.
5. The hardest or highest-risk requirement.
6. Usually no. State what changes, preserve the existing parts that still work, and evolve only the affected area.

</details>

## Interview Questions

You should answer each in 30–90 seconds.

1. What are the first questions you ask in a system-design interview?
2. Why do capacity estimates matter?
3. When should you choose the database?
4. What is the purpose of a deep dive?
5. How do you handle a requirement change?
6. What does a good trade-off statement sound like?
7. What should your closing summary contain?
8. What is the most common failure mode of an interview structure?

## Common Interview Mistakes

- Designing before clarifying the actual product.
- Asking dozens of low-value requirement questions.
- Calculating storage/QPS without tying numbers to decisions.
- Choosing technologies by brand recognition.
- Drawing a large architecture before defining the core data/flows.
- Never narrating the read/write path.
- Treating every requirement as equally important.
- Ignoring failure and retries.
- Adding microservices, Kafka, Redis, Elasticsearch, and multiple databases with no need.
- Running out of time without a summary.

## Must Remember

- **Requirements drive the architecture.**
- **Estimate only what changes a design decision.**
- **Access patterns drive the data model.**
- **Start with the simplest architecture that satisfies the requirements.**
- **Deep-dive into the hardest requirement, not your favorite technology.**
- **Every major component needs a reason.**
- **Every major choice has a trade-off.**
- **Discuss slow dependencies, retries, duplicates, and overload—not only total failures.**
- **Requirement changes should evolve the design, not destroy your structure.**
- **Finish with a concise summary.**

## Interview Revision Summary

Use this mental script:

```text
1. Clarify core functionality and non-goals.
2. Rank latency/availability/durability/consistency/throughput.
3. Estimate useful peak workload.
4. Define core APIs/events and access patterns.
5. Model source-of-truth data.
6. Draw the simplest working architecture.
7. Narrate one read and one write.
8. Deep-dive into the hardest requirement.
9. Inject failure, skew, retries, and 10× load.
10. State trade-offs and close.
```

## Explain Without Notes

Explain the full interview flow in two minutes. Then choose one system—chat, feed, ticket booking, or file storage—and say what you would deep-dive into and why.

## Completion Checklist

- [ ] I can lead the interview flow without notes.
- [ ] I can distinguish functional and non-functional requirements.
- [ ] I can connect estimates to architectural decisions.
- [ ] I can identify the correct deep-dive area.
- [ ] I can state trade-offs rather than only naming technologies.
- [ ] I can adapt to a requirement change.
- [ ] I can finish with a two-minute summary.
