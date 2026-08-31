# Session 44 — Timed Mock #4 — Trending Topics

## Timed Mock Instructions

**Time:** 45–55 minutes  
**Notes:** None  
**Solution lookup:** Not allowed  
**Goal:** Lead the interview aloud while drawing.

### Prompt

> Design a global trending-topics service that shows the most popular topics over recent time windows.

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

High scale, skew/hot keys, streaming aggregation, approximate algorithms, multi-region merge, cache.

### Information You May Assume Initially

- Input events come from posts/searches/clicks.
- Need top topics for 5-minute and 1-hour windows.
- Results can be a little approximate.
- Users see global and regional trends.
- Trend results should refresh frequently.

---

# STOP SCROLLING

Start your timer now.

Do **not** open the sections below until the appropriate point in the mock.

<details>
<summary><strong>INTERVIEWER PACK — Open only after your initial architecture</strong></summary>

### Clarifications
- Millions of events/sec globally.
- Topic extraction is already done; events contain normalized topic IDs.
- Exact counting is not required.
- Top results should update within ~10–30 seconds.

### Requirement Change
> One topic suddenly accounts for 40% of all events worldwide.

Ask:
- what key/partition becomes hot?
- how do you split aggregation?
- can local partial counts be merged?

### Failure Scenario
> One region loses connectivity to the global aggregation layer for ten minutes.

Ask:
- can regional trends continue?
- what does global trend show?
- how are late aggregates merged on recovery?

### 10× Push
> Which component breaks first at 10× event volume, and how do you evolve it?

</details>

<details>
<summary><strong>POST-MOCK REVIEW SIGNALS — Open only after the timer ends</strong></summary>

These are not a single canonical solution. Use them to detect whether you missed important reasoning.

Strong signals:

- local/regional aggregation before global merge to reduce raw cross-region bandwidth;
- time windows and late-event policy;
- partition/shard by topic plus additional salt when a topic becomes hot;
- partial counters can be merged;
- approximate heavy-hitter/top-K algorithms may be mentioned conceptually, but exact algorithm implementation is unnecessary;
- global result is derived data and can be stale;
- cached top-K response is ideal for reads;
- region can continue regional trends during global partition;
- global view may omit/stale that region and later merge/recompute;
- stream lag, hot-partition metrics, freshness SLO;
- bot/abuse manipulation and trust weighting/rate limits;
- no global strong consistency required.

Critical warning signs:
- one counter key per topic with no hot-key strategy;
- shipping all raw global events to one central machine;
- exact globally coordinated counter requirement invented unnecessarily;
- no window/late data semantics;
- cache used as authoritative event store.


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
