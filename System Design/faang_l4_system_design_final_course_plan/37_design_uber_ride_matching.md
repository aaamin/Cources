# Session 37 — Design Uber / Ride Matching

**Phase:** Phase 3 — Advanced System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Combine high-frequency location updates, geospatial candidate search, ephemeral state, durable trips, and regional failure.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Separate driver location/presence from durable trip/order records.
- Location ingestion at high write frequency.
- Geohash/grid/quadtree concepts and nearby-cell search.
- Candidate generation versus ranking/matching.
- Freshness TTL and stale drivers.
- Realtime updates to rider/driver.
- Regional partitioning by city/geography.
- Hot city/stadium load and dense cells.
- Matching/idempotency so one driver is not assigned inconsistently.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design nearby-driver lookup first, then trip matching.
- [ ] Change request: stadium empties during degraded regional connectivity.
- [ ] Explain how stale location data is detected and excluded.

## **Must Remember for the Interview**

- **Ephemeral driver location and durable trip state have different storage/consistency needs.**
- **Spatial indexing narrows candidates; exact distance/ranking can happen afterward.**
- **Location freshness matters as much as location accuracy.**
- **Dense cells and major events create hotspots.**
- **Trip assignment needs stronger correctness than location updates.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Driver updates → regional location service/index.**
- **Rider query → nearby spatial cells → candidates → filter/rank → match.**
- **Use TTL/heartbeat for stale drivers.**
- **Persist trip state separately with stronger invariants/idempotency.**
- **Plan for city-level hotspots and regional degradation.**

## Self-Test Before Marking This Session Complete

- [ ] Did I separate ephemeral and durable state?
- [ ] Did I explain spatial indexing?
- [ ] Did I handle stale drivers?
- [ ] Did I address hotspot/dense-cell behavior?
- [ ] Did I preserve correct trip assignment?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** A stadium empties during degraded regional connectivity.

**Score using the 40-point rubric.**


---

**Progress:** Session 37/46  
**Next:** Session 38
