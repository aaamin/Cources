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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 38 — Design Ticketmaster
