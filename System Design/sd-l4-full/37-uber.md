# Lesson 37 — Design Uber / Ride Matching

**Phase:** Advanced Design  
**Session:** 37/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- Drivers publish frequent locations.
- Riders find nearby available drivers.
- Create a durable ride assignment/trip.
- Handle hot cities, stale positions, and degraded connectivity.

## 2. Scale and workload shape

Estimate active drivers, location updates/driver/sec, ride requests/sec, geographic distribution, acceptable location freshness, and peak city events. The location path is high-write and ephemeral; trip state is lower-volume but correctness-sensitive.

## 3. API / contract surface

```http
POST /v1/drivers/location
POST /v1/rides
GET  /v1/rides/{id}
```

Live driver/rider updates can use WebSockets or streaming channels after a trip is established.

## 4. Data model

Separate ephemeral from durable state:

```text
DriverLocation(driver_id, lat, lon, updated_at, availability)  # fast/TTL
Trip(trip_id, rider_id, driver_id, state, pickup, dropoff, ...) # durable
```

A driver should not be matchable if location/availability is stale.

## 5. High-level architecture

```text
Driver App → Location Ingest → Geo Index / Fast Store
                                      ↑
Rider App → Ride Service → Nearby Search → Matching
                                ↓
                             Trip DB
                                ↓
                       realtime trip channel
```


Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Spatial indexing

Use geohash/grid/quadtree concepts to map coordinates into cells. Query the rider's cell plus neighbors, then refine candidate distance/ETA. Dense cells may need subdivision.

### Freshness

Location updates use TTL. If `now - updated_at` exceeds a threshold, do not present/match that driver.

### Matching

Separate candidate generation from ranking. First find nearby available drivers; then score ETA, acceptance probability, fairness, vehicle type, etc.

### Assignment correctness

A driver must not accept two rides concurrently. Use atomic state transition/conditional update on durable assignment state.

## 7. Failure modes and recovery

- Hot stadium cell: subdivide or shard high-density cells; batch updates.
- Driver disconnect: TTL removes them from available set.
- Duplicate ride request: client idempotency key returns same logical trip.
- Matching service retries: assignment CAS prevents double-assignment.
- Regional degradation: keep durable trips in home region and reduce search radius/cross-region dependence.
- Location store down: existing trips continue; new matching may degrade/unavailable.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Location can be eventually consistent and ephemeral; trip assignment/payment require stronger correctness. Geo partitioning gives locality but complicates cross-border/cross-region movement.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Location update path

Drivers may send GPS every few seconds. Writing every sample into the durable trip database creates unnecessary write load. Keep the latest location in a geo-indexed fast store with TTL; optionally stream historical samples to analytics storage asynchronously.

```text
Driver update
→ authenticate + validate
→ write latest location/availability
→ update geo cell membership
→ optional telemetry event
```

### Nearby search

A rider query first finds nearby spatial cells, retrieves available driver candidates, rejects stale positions, and then computes more precise distance/ETA. Increasing search rings gradually is cheaper than scanning an entire city.

### Race between candidates

The same driver may appear as candidate for several riders. Candidate lookup is approximate; **assignment is authoritative**. Matching must atomically transition driver availability or trip assignment so only one ride wins.

### Hot geography

Downtown/stadium cells can contain thousands of drivers and riders. Adaptive subdivision, multiple buckets per dense cell, and regional/city sharding prevent one spatial key from dominating. Sparse rural areas can use larger cells.

### Trip state machine

```text
REQUESTED → MATCHING → DRIVER_ASSIGNED → ARRIVING
→ IN_PROGRESS → COMPLETED / CANCELLED
```

Durably storing valid transitions makes recovery and customer support possible. Realtime events are derived notifications about that state.

### Common interview mistakes

- Putting every GPS update in the transactional Trip DB.
- Treating nearest-neighbor result as final assignment without CAS/locking.
- Returning drivers with stale locations.
- Using one fixed geohash precision for both Manhattan and countryside.
- Assuming live WebSocket state is durable trip truth.

### Reusable patterns learned

Ephemeral vs durable state, spatial indexing, candidate generation vs authoritative assignment, TTL freshness, hot-region partitioning, and state-machine workflows.


## Detailed reference design

### Separate the two hard problems

Ride matching has:

1. **ephemeral high-frequency location state**;
2. **durable correctness-heavy trip state**.

Do not store 1 Hz driver GPS updates in the same transactional model as payments/trip history without a reason.

### Location ingest

Driver app sends:

```text
driver_id
lat/lon
heading/speed?
timestamp
availability
```

Regional ingest authenticates, validates freshness, and writes to a fast geo-indexed store. Entries have TTL; a driver with no heartbeat/location update becomes stale and unavailable.

### Spatial index mental model

Geohash/grid divides the map into cells. To find drivers near rider:

1. identify rider's cell;
2. query same + neighboring cells;
3. expand radius if not enough candidates;
4. calculate actual distance/ETA for candidates;
5. filter stale/busy drivers.

Dense downtown cells may need subdivision; rural areas need broader search.

### Candidate generation vs matching

Nearby lookup returns candidates. Matching applies business logic:

- ETA;
- vehicle type;
- driver availability;
- acceptance/fairness;
- surge/zone policy.

Keep spatial index focused on fast candidate retrieval rather than encoding every business rule.

### Trip state machine

```text
REQUESTED
  ↓
MATCHING
  ↓
DRIVER_ASSIGNED
  ↓
DRIVER_ARRIVING
  ↓
IN_PROGRESS
  ├→ COMPLETED
  └→ CANCELED
```

Durable Trip DB owns these transitions. Assignment must avoid two riders concurrently claiming the same driver.

Use conditional state change:

```text
assign driver only if driver availability_version still AVAILABLE
```

or a matching owner/lock with short scope.

### Realtime communication

Rider/driver status updates can use WebSockets/push. Durable state remains in Trip DB; live channel loss triggers reconnect + fetch current trip.

### Regional partitioning

Trips and location are naturally regional by city. Route users/drivers to nearby region. Cross-city/global queries are rare on hot path. This reduces latency and blast radius.

### Hot event: stadium exit

One geo cell may receive huge rider and driver updates. Mitigations:

- subdivide cell;
- spread ingest partitions by `(cell, hash(driver))`;
- maintain aggregate/candidate lists per subcell;
- admission/surge policy;
- prioritize matching over non-critical analytics.

## Failure walkthrough

### Stale driver location

TTL marks unavailable. Matching should require freshness threshold and optionally ping/confirm driver before assignment.

### Two matchers choose same driver

Authoritative driver/assignment state uses conditional write/version so one wins. Losing matcher retries with another candidate.

### Regional connectivity degraded

Keep local trip/matching operations within region where possible. Cross-region analytics can lag. If global control is unavailable, preserve in-progress trips and reduce new matching scope rather than corrupt state.

### Driver app disconnects after assignment

Trip remains durable. Rider sees stale/connection status; system may wait/reassign according to policy.

## Interviewer follow-ups

### “Why not query SQL by lat/lon?”

At small scale, spatial indexes in SQL can work. At high-frequency millions-of-driver updates, a specialized in-memory/distributed geo index gives lower write/query latency. Start simple if scale allows.

### “Geohash problem at cell boundary?”

Query neighboring cells and refine by exact distance. A point near an edge may be physically close to drivers in another cell.

### “Where is driver availability stored?”

Ephemeral candidate availability can live with location, but authoritative assignment transition must be protected by durable/conditional state to prevent double booking.

## Common interview mistakes

- Durable trip state mixed with raw GPS stream.
- Geo cell queried without neighbors.
- No stale-location TTL.
- Matching result not conditionally committed.
- Global database for all location updates with no regional locality.
- WebSocket status treated as authoritative trip state.

## Short revision note

**Ride pattern:** regional location ingest → spatial candidate lookup → business matcher → conditional durable assignment/trip state → realtime updates. Ephemeral location and durable trip state are separate.

## Topics to revise

- [ ] geohash/grid/quadtree concept
- [ ] neighbor-cell search
- [ ] freshness TTL
- [ ] candidate vs match
- [ ] trip state machine
- [ ] conditional driver assignment
- [ ] regional partitioning
- [ ] hotspot/stadium scenario

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll separate high-frequency ephemeral driver locations from durable trip state. Regional location ingest maintains a geo index with freshness TTL; nearby search generates candidates; matching conditionally assigns one driver; Trip DB owns lifecycle and realtime channels only reflect it.

## How the design evolves at 10×

At 10× city traffic, subdivide hot geo cells, partition ingest by cell+driver hash, and isolate regions. At 10× trips, Trip DB partitions by city/user/trip while maintaining single-owner assignment correctness.

## Quick revision flashcards

**Geo edge issue?**  
Query neighboring cells then refine exact distance.

**Stale driver?**  
TTL/freshness threshold excludes it.

**Double assignment?**  
Conditional authoritative state transition; only one matcher wins.

**GPS storage?**  
Fast ephemeral geo store, not necessarily durable trip DB.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 38 — Design Ticketmaster
