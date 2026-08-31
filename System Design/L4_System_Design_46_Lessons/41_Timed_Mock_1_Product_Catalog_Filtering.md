# Session 41 — Timed Mock #1 — Product Catalog & Filtering

## Timed Mock Instructions

**Time:** 45–55 minutes  
**Notes:** None  
**Solution lookup:** Not allowed  
**Goal:** Lead the interview aloud while drawing.

### Prompt

> Design a large e-commerce product catalog and filtering service.

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

A relatively straightforward **read-heavy/API/data-model** service. Establish a clean baseline. Do not overcomplicate it.

### Information You May Assume Initially

- Products have title, description, price, seller, category, inventory status, and attributes such as brand/size/color.
- Millions of products.
- Users browse categories and filter/sort.
- Product detail and browse traffic are read-heavy.
- Checkout itself is out of scope.

---

# STOP SCROLLING

Start your timer now.

Do **not** open the sections below until the appropriate point in the mock.

<details>
<summary><strong>INTERVIEWER PACK — Open only after your initial architecture</strong></summary>

### Interviewer Clarifications if the candidate asks
- Peak product-read traffic: assume tens of thousands of RPS.
- Product updates: thousands/sec across marketplace during peak ingestion.
- Filters differ by category; e.g. laptops have RAM/CPU while clothing has size/material.
- Search-by-keyword may be included after the baseline browse/filter design.

### Requirement Change — introduce around minute 30
> Sellers now update price and availability very frequently, and users should see price changes within a few seconds.

Ask:
- Which data is source of truth?
- Which derived indexes/caches may lag?
- How do updates reach the query index?
- How do you prevent a stale browse page from being used as authoritative checkout inventory?

### Failure Scenario — introduce around minute 40
> The search/filter index is unavailable for 20 minutes, but the product source database is healthy.

Ask:
- What degrades?
- Do product-detail reads still work?
- Is there a fallback for top categories?
- How is the index rebuilt/replayed?

### Push question
> Which part breaks first at 10× traffic?

</details>

<details>
<summary><strong>POST-MOCK REVIEW SIGNALS — Open only after the timer ends</strong></summary>

These are not a single canonical solution. Use them to detect whether you missed important reasoning.

A strong design usually notices:

- flexible category attributes make access-pattern modeling important;
- product source-of-truth and search/filter read index need not be the same store;
- a search engine/inverted/faceted index is reasonable only when filter/relevance requirements justify it;
- product detail can be cached heavily;
- browse/filter results can be cached by popular query combinations, but cache-key explosion must be considered;
- seller updates can flow through DB/outbox/CDC into the derived search index;
- derived index freshness can be eventual while checkout uses authoritative inventory/price validation;
- keyword search is a natural extension of the search index, not a reason to move transactional truth there;
- index outage should degrade discovery without necessarily taking down direct product lookup;
- rebuild/replay path matters;
- pagination should be bounded/cursor-based where appropriate;
- tenant/seller abuse and expensive query filters need limits;
- media belongs behind object storage/CDN.

Critical warning signs:
- Elasticsearch/search index declared source of truth without justification;
- arbitrary dynamic SQL for every filter;
- no distinction between display availability and checkout inventory;
- no failure path for index/cache.


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
