# Session 2 — Back-of-the-Envelope Estimation

## Outcome

You should be able to estimate workload, storage, bandwidth, and concurrency quickly enough to guide design choices without turning the interview into a math exercise.

## Why This Matters

System design is about building for a workload. “Use a cache” is meaningless until you know whether you have 20 RPS or 200,000 RPS. Estimates help answer questions such as:

- Can one database node plausibly handle this?
- Is sharding likely to be necessary?
- Is media bandwidth a dominant concern?
- Is a CDN valuable?
- How many persistent connections must gateways maintain?
- How quickly will storage grow?
- Is a rare burst more important than the average?

Interview estimates are intentionally rough. The goal is order-of-magnitude reasoning.

## Mental Model

Estimate in this order:

```text
Users
  ↓
Actions/user/time
  ↓
Average workload
  ↓
Peak/skew/burst
  ↓
Payload size
  ↓
Storage/bandwidth/connections
  ↓
Architecture implication
```

## Useful Units

Keep common conversions in your head:

```text
1 day ≈ 86,400 seconds ≈ 100,000 seconds for rough math
1 KB ≈ 10^3 bytes
1 MB ≈ 10^6 bytes
1 GB ≈ 10^9 bytes
1 TB ≈ 10^12 bytes
```

For interview math, powers of ten are often sufficient.

## Requests Per Second

Suppose:

- 10M DAU;
- each user creates 2 writes/day;
- each user performs 20 reads/day.

Writes/day:

```text
10M × 2 = 20M writes/day
20M / 100k seconds ≈ 200 average write RPS
```

Reads/day:

```text
10M × 20 = 200M reads/day
200M / 100k ≈ 2,000 average read RPS
```

Now apply a peak factor. If traffic is human-driven, a 3×–10× peak may be reasonable depending on the product.

If we assume 5×:

```text
Peak reads ≈ 10k RPS
Peak writes ≈ 1k RPS
```

Do not pretend the peak factor is known. State the assumption.

## Read/Write Ratio

The ratio can affect architecture.

```text
Read-heavy:
cache
read replicas
precomputation
CDN

Write-heavy:
partitioning
batching
append-oriented storage
backpressure
stream processing
```

But ratios do not mechanically dictate a technology. A strongly consistent write-heavy workload may still need relational transactions.

## Storage Growth

Suppose a photo service receives:

- 2M photos/day;
- average compressed photo = 2 MB.

Daily storage:

```text
2M × 2 MB = 4M MB ≈ 4 TB/day
```

One year without deletion:

```text
4 TB × 365 ≈ 1.46 PB/year
```

That immediately tells you media should live in object storage, not as blobs in the primary relational database.

Then separately estimate metadata, which may be tiny in comparison.

## Bandwidth

Bandwidth is throughput of bytes, not requests.

Suppose playback traffic is:

- 20k peak requests/s;
- average response transferred = 500 KB.

```text
20,000 × 500 KB
≈ 10,000,000 KB/s
≈ 10 GB/s
≈ 80 Gb/s
```

That is a strong signal for CDN/edge delivery and origin protection.

## Concurrent Users and Connections

Persistent connection systems require a different estimate.

Suppose:

- 5M simultaneously online users;
- each keeps one WebSocket connection.

You need capacity for roughly 5M active connections regardless of whether each user sends only one message every few minutes.

The bottleneck may become:

- file descriptors;
- memory per connection;
- network bandwidth;
- gateway fan-out;
- connection rebalancing after failures.

## Retention

Storage growth depends on how long data is retained.

Examples:

- logs for 7 days vs 1 year;
- chat forever vs 90 days;
- raw metrics for one month, downsampled aggregates for one year.

Retention often changes the storage tier and cost more than QPS does.

## Peak, Burst and Skew

Average traffic is often misleading.

Three different effects:

### Peak
Normal daily high-water mark.

### Burst
Very short sudden increase—e.g. celebrity post, ticket sale, emergency alert.

### Skew
Traffic is uneven across keys/tenants/regions.

A system at only 20% average capacity can still fail because one shard receives most of the traffic.

## Capacity Headroom

If expected peak is 1,000 RPS and you want 30% headroom:

```text
Required capacity ≈ 1,300 RPS
```

Be clear whether “30% headroom” means 30% above forecast or running at 70% utilization. Interviewers usually accept either if you define it.

## Estimate the Dominant Thing

Not every design needs all calculations.

For URL shortener:
- read/write QPS;
- storage for mappings;
- cache implications.

For WhatsApp:
- concurrent connections;
- messages/sec;
- message retention.

For YouTube:
- video upload storage;
- playback bandwidth;
- transcoding workload.

For Ticketmaster:
- peak burst;
- contention on inventory;
- number of seats/events.

## Architecture Implications

A useful estimate is followed by a sentence like:

> At 50k peak read RPS, I would horizontally scale stateless application servers and strongly consider caching; I would not automatically shard the database unless the database workload shows that one node/replica topology is insufficient.

This is stronger than:

> 50k RPS means use Cassandra.

## Worked Example — Notification Service

Assume:
- 20M users;
- average 5 notifications/user/day;
- 20% of notifications happen in a 2-hour morning window;
- each queue message is ~1 KB.

Total/day:

```text
100M notifications/day
```

Average:

```text
100M / 100k ≈ 1k notifications/s
```

Morning 20%:

```text
20M notifications / 7,200 sec ≈ 2.8k/s
```

If a campaign can trigger 10M notifications in 10 minutes:

```text
10M / 600 ≈ 16.7k/s
```

Now the burst is much more important than the average. The queue should absorb the campaign, workers should be rate-controlled by provider limits, and password resets may need a separate high-priority path.

## Small Design Drills

1. 100M DAU, 10 reads/user/day. Rough average RPS?
2. A system stores 5M 10-KB objects/day. Rough daily storage?
3. A chat app has low message RPS but 30M concurrent users. What may dominate?
4. Why can 99% of traffic coming from 1% of users invalidate a simple average?
5. If a database handles 5k QPS today and projected peak is 4k QPS, are you done?
6. A video response is 5 MB while a metadata response is 2 KB. Which estimate is more likely to dominate network cost?

<details>
<summary>Answer key</summary>

1. 1B reads/day / ~100k ≈ 10k average RPS.
2. 5M × 10 KB = 50M KB ≈ 50 GB/day.
3. Persistent connection capacity: memory/file descriptors/network/gateway capacity.
4. Hot users/keys can overload individual partitions or caches even if total capacity is fine.
5. Not necessarily. Consider headroom, bursts, failure capacity, write mix, latency targets, and growth.
6. Video/media bandwidth.

</details>

## Common Mistakes

- Using average QPS as capacity.
- Calculating to false precision.
- Estimating every possible metric.
- Forgetting read/write ratio.
- Ignoring traffic skew.
- Ignoring retention.
- Confusing bandwidth with request rate.
- Forgetting concurrent connections for realtime systems.
- Jumping from a rough estimate directly to a named database.

## Must Remember

- **Use order-of-magnitude math.**
- **State assumptions.**
- **Peak and skew matter more than average for failure risk.**
- **Retention drives storage growth.**
- **Persistent connections require concurrency estimates, not only RPS.**
- **Payload size converts request rate into bandwidth.**
- **Every estimate should produce a design implication.**
- **Do not shard only because a number sounds large.**

## Interview Revision Summary

Core formulas:

```text
Average RPS ≈ requests/day ÷ 100,000
Storage/day = objects/day × bytes/object
Bandwidth = requests/sec × bytes/response
Concurrent connections ≈ simultaneously online users × connections/user
```

Always ask:
- peak factor?
- skew?
- burst?
- retention?
- headroom?
- what decision changes because of this number?

## Explain Without Notes

Estimate a hypothetical photo-sharing workload with:
- 10M DAU;
- 5 feed reads/day;
- 0.2 photo uploads/day;
- 2 MB average image;
- 5× peak.

Do the rough math and state at least three architecture implications.

## Completion Checklist

- [ ] I can estimate average and peak RPS quickly.
- [ ] I can estimate storage growth.
- [ ] I can estimate bandwidth.
- [ ] I remember to consider concurrent connections.
- [ ] I consider skew/burst/headroom.
- [ ] I state architectural implications after estimates.
