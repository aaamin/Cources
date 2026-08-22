# Lesson 43 — Mock #3 — Data-Heavy / Write-Heavy System

**Course:** FAANG L4 System Design Interview Course Plan — V2  
**Session:** 43 / 46  
**Phase:** Mock Interview  
**Recommended time:** 45–55 minute mock + 30–45 minute review

---

# STOP BEFORE READING FURTHER

This file contains the mock prompt.

When you are ready to start:

1. set a **55-minute maximum timer**;
2. prepare your diagramming surface;
3. do not open reference material;
4. then continue.

The goal is interview performance, not studying the prompt beforehand.

---

## 1. Mock Prompt

> **Design an IoT Telemetry Platform**

Millions of devices send measurements continuously. Users query recent data and hourly aggregates. Data should be retained according to tiered policies.

Do not attempt to cover every possible feature. Clarify scope as if an interviewer were present.

---

## 2. Interview Conditions

You must:

- speak while designing;
- clarify requirements;
- state non-goals;
- estimate only decision-relevant scale;
- define APIs/events and data ownership;
- start with a simple complete architecture;
- narrate at least one important flow;
- deep-dive the hardest requirement;
- discuss failures and trade-offs;
- finish with a concise summary.

Do not use notes or a known solution.

---

## 3. Mid-Interview Requirement Change

At approximately minute 25–30, introduce:

> **Halfway through, add replay of historical events into a new aggregation job.**

Adapt the existing design. Do not erase everything and start over.

---

## 4. Failure Scenario

At approximately minute 40, introduce:

> **One tenant begins sending 20× its normal traffic.**

Explain:

```text
User-visible impact
Detection
Immediate behavior
Correctness implications
Recovery
Follow-up design improvement
```

---

## 5. Suggested Time Budget

| Time | Activity |
|---:|---|
| 0–6 min | Requirements and scope |
| 6–10 min | Scale |
| 10–16 min | APIs/events/data model |
| 16–28 min | High-level architecture and flows |
| 28–42 min | Deep dive + requirement change |
| 42–49 min | Failure/scaling/production review |
| 49–52 min | Summary |

You may deviate if the interviewer redirects you, but maintain control of the remaining time.

---

## 6. 40-Point Rubric

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


### Pass rules

A strong mock should reach **32/40 or higher**.

A 0 or 1 in any of these is a non-pass regardless of total:

- Requirements
- APIs/Data
- High-Level Design
- Correctness
- Trade-Offs
- Communication

---

## 7. Self/Peer Interviewer Follow-Ups

Use 3–5 after the main design.

- How does your design handle **unseen prompt**?
- How does your design handle **partitioning**?
- How does your design handle **backpressure**?
- How does your design handle **replay/recovery**?
- How does your design handle **retention**?

Additional generic follow-ups:

- What fails first at 10× traffic?
- Where can duplicate work occur?
- Which operation needs the strongest consistency?
- What is your source of truth?
- How do you detect overload before users complain?
- What would you simplify for a first version?
- What would you change for another region?

---

## 8. Review

After the mock, do not immediately repeat the entire problem.

First score it.

```text
Requirements & Scope:       /4
Estimation & Workload:      /4
APIs / Events / Data:       /4
HLD & Flows:                /4
Scalability & Performance:  /4
Correctness & Consistency:  /4
Reliability & Operations:   /4
Security / Privacy / Cost:  /4
Trade-Offs & Evolution:     /4
Communication & Time:       /4

TOTAL:                      /40
```

Then record:

```text
Two weakest categories:
1.
2.

Three biggest misses:
1.
2.
3.

Requirement change response:
What went well?
What went poorly?

Failure response:
What went well?
What went poorly?
```

---

## 9. Targeted Repair

For each weak category:

1. perform 2–3 narrow drills;
2. redraw only the weak section from scratch;
3. explain it aloud;
4. do not memorize the whole architecture.

Examples:

### Weak data model

List access patterns → model entities → select indexes → state invariants.

### Weak reliability

Inject slow dependency → timeout → retry → duplicate → overload → recovery.

### Weak communication

Practice a 90-second opening and a 2-minute closing.

---

## 10. Completion Checklist

- [ ] Mock was unseen before starting.
- [ ] I stayed within 55 minutes.
- [ ] I clarified requirements and non-goals.
- [ ] I made decision-relevant estimates.
- [ ] I defined APIs/events/data.
- [ ] I completed an end-to-end architecture.
- [ ] I narrated main flows.
- [ ] I handled the requirement change.
- [ ] I handled the failure scenario.
- [ ] I gave a closing summary.
- [ ] I scored all ten rubric categories.
- [ ] I identified and repaired the two weakest categories.

---

## 11. Readiness Rule

The course is not complete merely because you reached Lesson 46.

You are considered ready when:

- the **latest three unseen mocks each score at least 32/40**;
- no category is below 2;
- requirements, APIs/data, HLD, trade-offs, and communication average at least 3;
- you consistently finish within time;
- you can absorb a requirement change and failure scenario without losing structure.

If the gate is missed:

> repair the weakest category and run another unseen mock.

Do not restart the course.

---

**Next:** Lesson 44 — Mock #4 — High Scale with Skew
