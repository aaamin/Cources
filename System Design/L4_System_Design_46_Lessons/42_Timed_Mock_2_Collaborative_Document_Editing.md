# Session 42 — Timed Mock #2 — Collaborative Document Editing

## Timed Mock Instructions

**Time:** 45–55 minutes  
**Notes:** None  
**Solution lookup:** Not allowed  
**Goal:** Lead the interview aloud while drawing.

### Prompt

> Design a collaborative document-editing system where multiple users can edit the same document and see updates in near real time.

### Your responsibilities

You must:
1. clarify functional requirements;
2. prioritize non-functional requirements;
3. estimate only design-relevant workload;
4. define APIs/events and core data;
5. draw a simple high-level architecture;
6. narrate one main read and one main write;
7. deep-dive into the hardest requirement;
8. respond to interviewer changes;
9. handle one failure scenario;
10. close with trade-offs and 10× evolution.

### Focus

Realtime communication, concurrent edits, connection management, durability, and conflict semantics.

### Information You May Assume Initially

- Think Google Docs-like text collaboration at a high level.
- Users can open a document, edit, reconnect, and see collaborators.
- Focus on text operations and synchronization, not rich media.
- Authentication and document sharing exist.

---

# STOP SCROLLING

Start your timer now.

Do **not** open the sections below until the appropriate point in the mock.

<details>
<summary><strong>INTERVIEWER PACK — Open only after your initial architecture</strong></summary>

### Clarifications
- Up to hundreds of simultaneous editors on a very hot document.
- Millions of documents overall.
- Users may go offline briefly and reconnect.
- Update latency target is sub-second for active collaborators.
- The interviewer does not require you to implement CRDT/OT internals. Recognition and system-level placement are enough.

### Requirement Change
> A globally distributed team edits one document from three continents. Users want low-latency local typing feedback.

Ask:
- Is one document assigned a home region/leader?
- What is local optimistic UI vs authoritative operation ordering?
- What does cross-region coordination cost?
- Would you accept eventual merge semantics or serialize document operations?

### Failure Scenario
> The realtime gateway handling 100k connections crashes.

Ask:
- How do clients reconnect?
- How do they discover missed operations?
- Which state is durable vs ephemeral?

### Correctness Push
> Two users edit the same text range concurrently.

The candidate should define a conflict/order model rather than hand-wave “WebSocket ordering.”

</details>

<details>
<summary><strong>POST-MOCK REVIEW SIGNALS — Open only after the timer ends</strong></summary>

These are not a single canonical solution. Use them to detect whether you missed important reasoning.

Strong signals include:

- WebSockets or similar for bidirectional active sessions;
- gateway/connection state separated from durable document state;
- document operations have IDs/versions/sequences;
- server stores an append/change log and periodic snapshots so reconnect does not replay entire history forever;
- optimistic client edits are distinct from authoritative merge/order;
- one document can have a collaboration/session leader or partition to serialize operations if using an OT-like centralized model;
- CRDT/OT can be mentioned at recognition depth, not implemented;
- hot documents may need dedicated collaboration partition/resources;
- presence/cursor positions are ephemeral and can be dropped/coalesced;
- durable text operations cannot simply be dropped;
- reconnect resumes from version/cursor;
- gateway failure causes reconnect storm, not document loss;
- document access/permissions need server authorization;
- snapshots, retention, compaction, and operation-log growth deserve mention.

Critical warning signs:
- “TCP/WebSocket gives ordering, so concurrent edits are solved”;
- no durable operation/version model;
- storing every cursor movement forever;
- no hot-document strategy;
- no reconnect/resync.


</details>

## 40-Point Scorecard

Score 0–4 in each category:

| Category | Score |
|---|---:|
| Requirements & Scope | /4 |
| Estimation & Workload | /4 |
| APIs / Events / Data Model | /4 |
| High-Level Design & Flows | /4 |
| Scalability & Performance | /4 |
| Correctness & Consistency | /4 |
| Reliability & Operations | /4 |
| Security / Privacy / Cost | /4 |
| Trade-Offs & Evolution | /4 |
| Communication & Time Control | /4 |
| **Total** | **/40** |

## Repair Rule

After scoring:

1. Identify the bottom two categories.
2. Write the single most damaging mistake in each.
3. Perform 2–3 narrow drills.
4. Redo only the weakest 15–20 minutes from a blank page.
5. Do not memorize a “perfect architecture.”

## Mock Completion Record

```text
Date:
Duration:
Score:
Bottom category #1:
Bottom category #2:
Best decision:
Biggest miss:
Requirement-change response:
Failure-scenario response:
What I will repair:
```
