# Lesson 25 — Object Storage, Media, Multi-Region & Disaster Recovery

**Course:** FAANG L4 System Design Interview Course Plan — V2  
**Session:** 25 / 46  
**Phase:** Fundamentals  
**Recommended time:** 60–90 minutes

---

## 1. Lesson Goal

This lesson builds one part of the vocabulary you will later combine in full system-design interviews.

By the end, you should be able to explain the core ideas without notes, apply them to a small backend scenario, identify important failure modes, and discuss at least one reasonable alternative.

### Core topics

- **Blob/Object Storage** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Metadata Vs Objects** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Multipart/Resumable Upload** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Checksums** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Presigned Urls** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Versioning** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Active/Passive Vs Active/Active** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Global Routing** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Rpo/Rto** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Backups Vs Replicas** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.
- **Cost And Egress Awareness** — explain what problem it solves, how it works conceptually, when to use it, and its trade-offs.

---

## 2. Why This Matters in an L4 Interview

At L4, naming a technology is not enough. Your answer should normally follow this reasoning chain:

```text
Requirement / workload
        ↓
Needed capability
        ↓
Design choice
        ↓
Trade-off
        ↓
Failure behavior
```

For this lesson, practice sentences such as:

> “Because the workload requires ___, I would use ___; the main trade-off is ___, and if it fails I would ___.”

The goal is not encyclopedic depth. It is dependable engineering judgment.

---

## 3. Core Concepts

### 1. Blob/Object Storage

For interview purposes, understand **blob/object storage** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 2. Metadata Vs Objects

For interview purposes, understand **metadata vs objects** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 3. Multipart/Resumable Upload

For interview purposes, understand **multipart/resumable upload** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 4. Checksums

For interview purposes, understand **checksums** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 5. Presigned Urls

For interview purposes, understand **presigned URLs** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 6. Versioning

For interview purposes, understand **versioning** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 7. Active/Passive Vs Active/Active

For interview purposes, understand **active/passive vs active/active** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 8. Global Routing

For interview purposes, understand **global routing** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 9. Rpo/Rto

For interview purposes, understand **RPO/RTO** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 10. Backups Vs Replicas

For interview purposes, understand **backups vs replicas** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?

### 11. Cost And Egress Awareness

For interview purposes, understand **cost and egress awareness** at conceptual depth.

Ask:

- What problem does it solve?
- What assumption does it make?
- What gets faster, safer, or easier?
- What becomes more complex or expensive?
- What happens when it is unavailable or wrong?
- What alternative could work for a smaller system?


---

## 4. Worked Scenario

**Scenario:** Design a resumable 10GB upload and then explain behavior during a complete regional outage.

Work through it using this sequence:

```text
1. State the requirement.
2. Identify the relevant workload characteristic.
3. Apply today's concept.
4. Explain the simplest acceptable design.
5. Describe one failure mode.
6. Give one alternative and trade-off.
```

Write a short answer before reading further material.

### Example answer shape

```text
Requirement:
Workload:
Choice:
Why:
Failure mode:
Mitigation:
Alternative:
Trade-off:
```

Do not worry about creating a complete system architecture; this is a focused fundamentals drill.

---

## 5. Interview Reasoning Checklist

When today's concept appears in a future design, ask:

- What exact problem am I solving?
- Does the current scale actually require this?
- What is the source of truth/state owner?
- What happens under peak load or skew?
- What happens when this component becomes slow?
- What happens when it becomes unavailable?
- Is there stale, duplicated, reordered, or partially completed state?
- What operational signal would reveal trouble?
- What simpler alternative exists?

---

## 6. Small Drills

### Drill A — Explain Without Product Names

Explain today's topic without naming Redis, Kafka, Cassandra, Nginx, Kubernetes, or any other vendor/product unless the concept genuinely requires an example.

Your answer:

```text

```

### Drill B — Use vs Don't Use

Write one situation where today's technique is a good fit and one where it is unnecessary or harmful.

```text
Good fit:

Poor fit:
```

### Drill C — Failure Injection

Assume the component or mechanism introduced today becomes:

1. slow;
2. unavailable;
3. overloaded.

For each case, describe the user-visible impact and one mitigation.

### Drill D — 10× Growth

Ask:

> “If the workload becomes 10× larger, which assumption in today's design fails first?”

Write the answer in 3–5 sentences.

---

## 7. Knowledge Check

Answer without looking at notes.

1. What primary problem does **Object Storage, Media, Multi-Region & Disaster Recovery** solve?
2. Which workload characteristics make it useful?
3. What is one common misuse or overengineering mistake?
4. What is one important failure mode?
5. What is one operational metric or symptom you would watch?
6. What is one simpler alternative?
7. What is one scalability trade-off?
8. What is one correctness or consistency concern?
9. How would you explain this concept in under one minute?
10. How might it appear in a real system design such as chat, feed, storage, or booking?

### Suggested scoring

- **8–10:** ready to move on.
- **6–7:** review weak concepts, then explain aloud again.
- **0–5:** repeat the core material before moving forward.

---

## 8. Spoken Exercise

Without notes, explain today's lesson for **3–5 minutes**.

Your explanation should include:

```text
Problem
How it works
When to use it
When not to use it
Failure mode
Trade-off
Example
```

If you cannot do this clearly, the session is not complete.

---

## 9. Completion Checklist

- [ ] I can explain the lesson's main problem without notes.
- [ ] I can explain each core topic at conceptual depth.
- [ ] I can apply the ideas to the worked scenario.
- [ ] I can identify at least two failure modes.
- [ ] I can give a simpler alternative.
- [ ] I can explain one major trade-off.
- [ ] I completed the drills.
- [ ] I scored at least 8/10 on the knowledge check.
- [ ] I gave a 3–5 minute spoken explanation.

---

## 10. Session Notes

### Concepts I understand well

```text

```

### Concepts still unclear

```text

```

### Three things to remember

1.
2.
3.

### Questions to revisit

```text

```

---

## 11. One-Page Recall Card

```text
LESSON 25: OBJECT STORAGE, MEDIA, MULTI-REGION & DISASTER RECOVERY

WHY?
- What problem does it solve?

HOW?
- What is the conceptual mechanism?

WHEN?
- What workload makes it useful?

TRADE-OFFS
- latency
- throughput
- correctness
- complexity
- cost

FAILURE
- slow
- unavailable
- overloaded

ALTERNATIVE
- What simpler design could work?

INTERVIEW RULE
- requirement → capability → design → trade-off → failure
```

---

**Next:** Lesson 26 — Observability, SLOs, Deployment, Security & Privacy
