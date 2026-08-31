# Session 37 — Advanced Design — Uber / Ride Matching

## Interview Prompt

> Design a ride-hailing system that lets riders request a ride, tracks online drivers, finds nearby candidates, matches one driver to one rider, and updates trip state.

### Mid-Interview Change

> A stadium empties at once. Ride demand and driver-location traffic spike sharply in one city while regional connectivity is degraded.

### Rules

Spend **45–55 minutes** on your own design before reading the reference section.

The interviewer is looking for:
- clear separation of durable vs ephemeral state;
- geospatial indexing;
- matching concurrency;
- hot-city handling;
- stale location;
- regional ownership/failure;
- trip/payment correctness.

---

# STOP — Complete Your Design First

Do not read further until you have answered:

1. How do driver locations enter the system?
2. How do you find nearby drivers?
3. How do you prevent the same driver from being assigned twice?
4. Which state must be durable?
5. What happens when a region is partially disconnected?
6. What breaks first at a stadium-scale burst?

---

# Interviewer Pressure Pack

Use these one at a time after your initial architecture.

### Pressure 1 — Location Skew

A city normally has 50k online drivers. A stadium event causes 200k riders to request within ten minutes from a small geographic area.

What becomes hot?

### Pressure 2 — Stale Driver

Driver A's last location is 45 seconds old but still appears available.

How do you avoid a terrible match?

### Pressure 3 — Double Assignment

Two matching workers concurrently choose the same driver.

How do you preserve:
> One driver can have at most one active ride assignment.

### Pressure 4 — Connectivity Degradation

Driver gateways can reach the regional matching service intermittently. Some acknowledgements are delayed or duplicated.

What states can become uncertain?

---

# Reference Reasoning

## 1. Requirements

Core:
- driver comes online/offline;
- driver sends location;
- rider requests ride;
- find nearby candidates;
- reserve/assign driver;
- rider/driver receive updates;
- trip state changes;
- cancellation;
- payment can be out of scope until trip completion.

Non-functional:
- low matching latency;
- location can be slightly stale;
- assignment must be correct;
- city/regional isolation;
- enormous skew around events;
- system should degrade rather than globally fail.

Define the critical invariant:

> **A driver must not be committed to two active rides at the same time.**

This is much stronger than “nearby search should be accurate.”

## 2. State Classification

### Ephemeral / high-frequency

```text
driver current location
driver online status
connection/gateway
candidate availability hints
```

Location can be lost/reconstructed from new updates.

### Durable

```text
ride request
accepted assignment
trip state
driver/rider IDs
fare/payment records
audit trail
```

Do not write every GPS update into the transactional ride database.

## 3. Driver Location Ingest

```text
Driver App
   ↓ persistent/mobile connection
Regional Gateway
   ↓
Location Ingest
   ↓
Geo Index / Ephemeral Store
   ↓
optional event stream for analytics/history
```

Update:
```text
driver_id
lat/lon
timestamp
availability_hint
sequence
```

The live matching index needs only recent location.

Historical telemetry can flow asynchronously to another storage pipeline.

## 4. Geospatial Partitioning

Possible conceptual representations:
- geohash/grid cells;
- quadtree;
- geospatial search index.

Grid mental model:

```text
city divided into cells
cell → available drivers
```

Nearby search:
1. inspect rider's cell;
2. expand neighboring cells until enough candidates;
3. rank by ETA/distance/driver suitability.

Raw Euclidean latitude/longitude scanning does not scale.

## 5. Hot Cells

Stadium:
- huge number of riders and drivers in a few cells.

One geo-cell key can become a hot partition.

Mitigations:
- smaller/adaptive cells;
- shard drivers within a cell;
- dedicated city cluster;
- read-replicate candidate sets;
- admission/waiting queue for rider requests;
- hierarchical spatial index.

Do not globally increase cell size: it can worsen candidate scans.

## 6. Candidate Search vs Assignment

Important separation:

### Search
Returns possible drivers. Can be based on slightly stale ephemeral data.

### Commit
Atomically reserves one driver.

Two matching workers can both discover Driver A. The commit layer decides winner.

Possible durable/strong operation:

```text
DriverAssignment(
  driver_id PRIMARY/UNIQUE active assignment
)
```

or state:

```text
AVAILABLE(version=41)
  ↓ compare-and-set
RESERVED(ride123, version=42)
```

Only one CAS/conditional transaction succeeds.

Do **not** attempt to make the entire geo index strongly consistent.

## 7. Matching Flow

```text
Rider request
  ↓
Ride Service persists REQUESTED
  ↓
Matching Service
  ↓
Geo index → candidate driver IDs
  ↓
score/filter stale candidates
  ↓
conditional reserve driver
  ├─ fail → try next
  └─ success
       ↓
persist assignment
       ↓
notify rider + driver
```

Driver may decline/timeout.

Reservation has:
- expiry/lease;
- ride ID;
- attempt number.

If reservation expires, driver returns available unless trip accepted.

## 8. Stale Location

Store update timestamp/sequence.

Reject or down-rank:
```text
now - last_location > threshold
```

Threshold can vary by city/speed.

Driver heartbeat/presence also matters.

Location freshness is a ranking/eligibility property, not necessarily a globally transactional value.

## 9. Driver Accept/Reject

Offer:
```text
driver reserved for ride123 for 10s
```

Driver accepts:
- conditional transition `RESERVED → ASSIGNED`.

Reject/timeout:
- release reservation;
- try next candidate.

Use idempotent command IDs because mobile retries.

## 10. Trip State Machine

Example:

```text
REQUESTED
 → MATCHING
 → DRIVER_ASSIGNED
 → DRIVER_ARRIVING
 → IN_PROGRESS
 → COMPLETED
```

Other:
```text
CANCELLED
```

State transitions should be validated/conditional.

Event duplicates should not move state backward.

## 11. Regional Model

Ride interactions are geographically local.

Natural partition:
```text
city/region
```

Each city has:
- local gateways;
- location index;
- matching workers;
- ride DB partition/home.

Benefits:
- low latency;
- failure isolation;
- traffic locality.

Cross-city/global user profile/payment can be separate.

## 12. Stadium Burst

Symptoms:
- rider requests spike;
- candidate searches hit same cells;
- notification traffic spikes;
- driver updates rise;
- surge calculation maybe hot.

Protect system:

### Admission / waiting
If matching capacity limited:
```text
REQUEST_RECEIVED → queue/waiting
```
Give rider progress rather than allowing all requests to time out.

### Separate location ingest and matching
Location updates should not be blocked by rider matching backlog.

### Prioritized queues
Active trip updates > new match retries > analytics.

### Dynamic cell partitioning
Hot area split or dedicated resources.

### Backpressure
Do not create unlimited matching retries.

## 13. Connectivity Degradation

Suppose driver receives offer and accepts, but acknowledgement path is flaky.

Possible:
- driver thinks accepted;
- server did not see response;
- server retries offer;
- reservation expires.

Use:
- durable assignment state;
- unique ride/offer IDs;
- idempotent accept;
- server authoritative state;
- reconciliation on reconnect.

Driver app reconnects and asks:
```text
GET current active assignment
```

Do not trust local app state as source of truth.

## 14. Regional Failure

If city region unavailable:
- can you fail over nearby region?
- geo state is ephemeral and must be rebuilt from reconnecting drivers;
- durable ride assignments/trips must replicate according to RPO;
- avoid two regions both becoming authoritative for driver assignment.

Safer active/passive:
- promote one secondary for city;
- fencing/epoch prevents old region from committing after recovery.

During severe partition, it may be safer to stop new assignments while preserving active-trip visibility than risk double assignment.

## 15. ETA/Route Service

Nearby distance is not equal to driving ETA.

Matching can call routing/ETA system for top N candidates.

Do not call expensive routing service for thousands of candidates:
1. geo filter cheaply;
2. select top ~N by approximate distance;
3. calculate accurate ETA.

Timeout/fallback if route service slow.

## 16. Observability

By city:
- location update age;
- online driver count;
- request rate;
- match latency;
- match success;
- candidate count;
- reservation conflict rate;
- waiting queue age;
- geo-cell hot spots;
- stale candidate rate;
- reconnects;
- regional replication health.

Average global metrics hide one city meltdown.

## 17. Security/Abuse

- authenticate driver devices;
- prevent fake location/spoof abuse where possible;
- rate limit location updates;
- protect rider/driver location privacy;
- minimize retention of precise location;
- authorize trip visibility.

## Alternative Designs

### Strong DB for all availability
Simple correctness, but high-frequency geo writes can overload transactional store.

### Ephemeral geo + strong reservation
Usually better separation:
- fast approximate discovery;
- strong narrow commit.

This “weak search, strong commit” pattern appears in many systems.

## Common Failure Answers to Avoid

- “Redis GEO solves matching.”
- “Use Kafka for locations” without realtime lookup.
- Strongly consistent GPS state everywhere.
- Same driver can be returned by concurrent workers with no commit guard.
- Global single matching cluster.
- Fail over region with no write-authority/fencing.
- Waiting queue with no age/UX.
- Location data kept forever with no privacy discussion.

## Must Remember

- **Separate ephemeral geo state from durable trip state.**
- **Nearby search may be approximate; assignment commit must enforce the invariant.**
- **Use conditional reservation/CAS to prevent double assignment.**
- **Partition naturally by city/region.**
- **Hot geographic cells are a skew problem.**
- **Stale location must be explicitly detected.**
- **Mobile retries/uncertain acknowledgements require idempotent state transitions.**
- **Regional failover must preserve one assignment authority.**
- **At event-scale overload, admission/backpressure is better than universal timeout.**

## Repair Exercise

Redo only the 15-minute deep dive for this scenario:

> 300k riders request from one airport district in five minutes. The route/ETA dependency becomes 5× slower and 10% of drivers have stale locations.

Your answer must preserve driver uniqueness and protect the route service.
