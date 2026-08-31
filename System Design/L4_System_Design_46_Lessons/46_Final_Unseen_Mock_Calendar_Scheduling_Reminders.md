# Session 46 — Final Unseen Mock — Calendar Scheduling & Reminders

## Timed Mock Instructions

**Time:** 45–55 minutes  
**Notes:** None  
**Solution lookup:** Not allowed  
**Goal:** Lead the interview aloud while drawing.

### Prompt

> Design a global calendar service supporting event creation, invitations, attendee responses, calendar views, recurring events, and reminders.

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

Final mixed-domain mock: APIs/data modeling, recurring semantics, concurrency, async reminders, multi-region, privacy, requirement changes.

### Information You May Assume Initially

- Users create events with start/end time and attendees.
- Invitees can accept/decline.
- Users view day/week/month calendar.
- Recurring events are supported.
- Reminders are delivered before events.
- Free/busy lookup is included.

---

# STOP SCROLLING

Start your timer now.

Do **not** open the sections below until the appropriate point in the mock.

<details>
<summary><strong>INTERVIEWER PACK — Open only after your initial architecture</strong></summary>

### Clarifications
- Tens of millions of active users.
- Most calendar reads are user/day-range queries.
- Event edits should appear quickly to attendees.
- Reminder delivery can be at-least-once but harmful duplicates should be minimized.
- Users exist across time zones.

### Requirement Change — introduce around minute 25
> An organizer edits only one occurrence of a recurring weekly meeting.

Ask:
- recurrence rule vs generated instances;
- exception/override modeling;
- what attendees see;
- reminder rescheduling.

### Requirement Change — around minute 35
> Add a “find a time when all 20 attendees are free” API.

Ask:
- free/busy data access;
- privacy;
- range query/indexing;
- fan-out/caching.

### Failure Scenario — around minute 43
> The reminder scheduler region is unavailable for 90 minutes. Some reminder times pass during the outage.

Ask:
- missed-reminder policy;
- durable schedule occurrence;
- duplicate prevention;
- recovery.

### Ambiguity Challenge
If the candidate assumes recurring events mean materializing infinite future rows, ask:
> How far into the future do you materialize?

### Final 10× question
> Which part changes first if enterprise tenants create calendars with millions of events?

</details>

<details>
<summary><strong>POST-MOCK REVIEW SIGNALS — Open only after the timer ends</strong></summary>

These are not a single canonical solution. Use them to detect whether you missed important reasoning.

Strong signals:

- Event/Calendar/Attendee/Response ownership is clearly modeled;
- range queries by calendar/user + start time drive indexes/partitioning;
- event ID stable across edits;
- recurring event stored as rule + bounded/materialized occurrences or generated-on-read hybrid;
- single-occurrence edit represented as exception/override rather than mutating whole series;
- time zone and DST semantics are explicit (local recurring time vs absolute UTC occurrence);
- invitations/updates use durable async events, idempotent processing;
- reminders are generated from durable schedule occurrences, claimed with lease/idempotency, and have explicit misfire policy after outage;
- free/busy exposes availability intervals rather than private event details;
- cache may help hot calendar ranges but authorization remains server-side;
- multi-region home region/user/calendar ownership is reasonable; global attendees receive replicated updates;
- conflict when two organizers/editors update event should use version/optimistic concurrency;
- large calendars need time partitioning/indexed range reads, not full scan.

Critical warning signs:
- infinite pre-generation of recurrence;
- UTC-only recurrence with no timezone/DST policy;
- reminders stored only in volatile queue;
- free/busy leaks event titles/details;
- duplicate reminder not considered;
- attendee updates synchronous fan-out to millions with no durable event pipeline.


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
