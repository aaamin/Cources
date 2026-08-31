# Lesson 02 — Back-of-the-Envelope Estimation

**Phase:** Fundamentals  
**Session:** 2/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn to estimate request volume, storage, bandwidth, concurrency, and peak/skew quickly enough to guide architecture. The purpose is not precision; it is to understand the workload shape and connect numbers to design choices.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Why estimation matters

A design for 100 requests/s can be very different from one for 100,000 requests/s. Estimation helps decide whether one database is enough, whether traffic is read-heavy or write-heavy, whether object storage matters, and whether persistent connections dominate. Use the pattern `estimate → architectural consequence`. If reads outnumber writes 100:1, caching and read scaling deserve more attention than write optimization.

## Requests per second

Start from daily volume when needed: `requests/day ÷ 86,400 ≈ average QPS`. Then apply an explicit peak factor. Consumer traffic is uneven, so average QPS is insufficient. A rough 3×–10× peak assumption is acceptable when stated clearly. The interviewer cares more about order of magnitude than arithmetic precision.

## Storage

Use `items/day × bytes/item × retention`. If 100M records/day average 1 KB, raw growth is about 100 GB/day before indexes, replication, metadata, and compression. Over a year that is tens of terabytes. Usually the important realization is whether you are dealing with GB, TB, or PB scale.

## Bandwidth

Bandwidth becomes critical when payloads are large. Approximate `requests/s × bytes/request`. A media system sending thousands of 1 MB responses each second should not push those bytes through application servers; CDN and object-storage delivery become natural.

## Concurrency

Some systems are limited by open connections rather than request rate. Chat may have millions of mostly idle WebSockets. Estimate concurrent users and connections/user because connection gateways consume memory, sockets, and routing state even when message QPS is modest.

## Peak and skew

Uniform traffic is rare. Flash sales, celebrities, hot products, and regional time zones create concentration. A shard holding one hot key can fail even when total system QPS looks safe. Estimate both total scale and distribution shape.

## Worked example — estimating a messaging workload

Assume 50M DAU and 30 messages/user/day. That is 1.5B messages/day, roughly 17k messages/s average. With a 5× peak, design around ~85k messages/s. At 500 bytes/message, raw storage grows about 750 GB/day. These numbers imply horizontal application capacity, a high-throughput message store, likely partitioning at scale, and a separate plan for tens of millions of concurrent connections.

## Interview lens

Keep estimation short. State assumptions, round aggressively, and say what each result changes. Avoid calculations that never influence the design.

## What to remember

Remember: QPS describes traffic, storage describes data volume, bandwidth describes data movement, concurrency describes open-state pressure, and skew describes where averages lie.

## Review after reading

1. Why is peak QPS more useful than average alone?
2. How do you estimate one year of storage?
3. When can concurrency matter more than QPS?
4. Give one skew example.
5. What could a 100:1 read/write ratio change?

## Deeper study notes

### Use powers of ten, not false precision

Interview estimation is a modeling exercise. If you assume 52 million users instead of 50 million, the architecture almost never changes. Round aggressively: 50M users, 100 bytes, 1 KB, 10× peak. The interviewer is looking for whether you can distinguish a service that fits on one machine from one that needs a distributed storage and delivery strategy.

A useful habit is to write the assumption and consequence together:

```text
100M feed reads/day → roughly 1k/s average, maybe 5–10k/s peak
10 KB/feed response → tens of MB/s, not a media-scale bandwidth problem
100:1 reads:writes → optimize the read path first
```

### Estimate the dominant resource

Different systems have different dominant resources. URL shortening is often lookup/QPS heavy. Video is bandwidth/egress heavy. Chat is connection-count + write heavy. Logging is storage/write throughput heavy. Ticketing is contention heavy. Estimation should reveal the dominant resource rather than produce every possible number.

### Peak, growth, and safety margin

If the interviewer gives current scale, also ask whether the design must support future growth. Designing exactly to today's average creates no room for traffic bursts, deployments, zone failure, or retry storms. A simple statement such as “I will provision for the stated peak plus operational headroom” is enough; you do not need a precise utilization target unless asked.

### Unit discipline

Many mistakes are unit mistakes. Keep conversions visible:

```text
1 KB ≈ 10^3 bytes
1 MB ≈ 10^6 bytes
1 GB ≈ 10^9 bytes
1 day = 86,400 seconds ≈ 10^5 seconds
```

Using `10^5 seconds/day` is convenient for fast mental math. If the result is approximate, say so.

### Common mistakes

- Estimating every possible metric even when none changes the design.
- Using only average traffic and ignoring flash peaks.
- Forgetting replication/index overhead when discussing storage capacity.
- Confusing requests per second with concurrent connections.
- Producing a number but never using it to justify a component.

### Interview-ready mental model

```text
Users/activity
   ↓
operations per user
   ↓
QPS + peak
   ↓
payload size
   ↓
bandwidth/storage
   ↓
connection count / skew
   ↓
architecture decision
```


## Important interview ideas

> **Important:** Estimation is useful only when it changes a decision. In an interview, every meaningful number should eventually answer a question such as “Do we need caching?”, “Do we need sharding?”, “Can one region handle this?”, or “Are media bytes the dominant cost?”

### Work from user behavior, not arbitrary infrastructure capacity

A reliable way to estimate is to begin with product behavior:

```text
users
× actions per user
× peak factor
× payload size
× retention
```

This is safer than beginning with assumptions such as “one server handles 10,000 QPS,” because server capacity depends heavily on language, workload, payload, database access, and hardware. First estimate **demand**. Capacity planning comes later.

For example, imagine a feed product with 20M DAU. If each user opens the feed 10 times/day, there are about 200M feed reads/day. Using `~10^5 seconds/day`, average QPS is roughly 2,000. If evening peak is 5×, design the read path for about 10,000 QPS plus operational headroom. That is enough to justify multiple app instances and perhaps caching, but it is nowhere near the bandwidth profile of video streaming.

### Separate request rate from data rate

Two systems can have the same QPS but completely different architecture because payload sizes differ.

| Workload | QPS | Approx payload | Main pressure |
|---|---:|---:|---|
| URL redirect | 100k/s | hundreds of bytes | lookup latency / cache |
| Image delivery | 100k/s | 500 KB | bandwidth / CDN |
| Log ingest | 100k/s | 1–5 KB | sustained write throughput |
| Chat heartbeat | 100k/s | tiny | connection/state overhead |

This is why “100k QPS” alone is not a full scale description.

### Storage estimates need overhead awareness

Raw payload is only the first approximation. Real storage may include:

- replication copies;
- secondary indexes;
- metadata;
- filesystem/object overhead;
- tombstones and old versions;
- compression;
- backups.

You do not need to calculate each precisely. A good interview statement is:

> “Raw storage is about 30 TB/year. With replication and indexes, I would expect the provisioned footprint to be materially larger, perhaps a few times that, so this is clearly beyond a single-machine storage plan.”

### Estimation and correctness are different

Do not let huge scale distract you from the actual hard problem. Ticket booking may have modest average QPS but extremely high contention around a few seats. Payment systems may be more constrained by correctness than throughput. A good designer recognizes the dominant difficulty rather than automatically turning every prompt into a throughput problem.

## Worked calculation set

### Example A — photo storage

Assume:

```text
5M uploads/day
average compressed photo = 2 MB
```

Raw new media:

```text
5M × 2 MB = 10 TB/day
```

At one year, that is multiple petabytes. The consequence is immediate: photos belong in object storage, not the relational metadata database. Delivery should use a CDN. The application database stores metadata and object references.

### Example B — notification throughput

Suppose 50M notifications/day. Average rate is only about 580/s, but an emergency campaign may send 10M notifications in 10 minutes:

```text
10M / 600 ≈ 16,700 notifications/s
```

The peak, not the daily average, drives queue and worker capacity. Provider quotas may become the real constraint.

### Example C — concurrent chat sockets

Suppose 100M DAU and 10% are concurrently online:

```text
10M concurrent users
```

Even if message QPS is only 50k/s, the gateway tier must maintain millions of sockets. That changes memory, network, routing, heartbeat, and deployment behavior.

## Interview questions and model answers

### Q1. “Do we always need to estimate storage?”

No. Estimate only if storage volume affects the architecture. For a rate limiter, state size and operation rate matter more than long-term storage. For Dropbox or YouTube, storage and bandwidth are central. I would say what I am skipping and why: “Storage is tiny for this service, so I’ll focus on QPS and atomic counter throughput.”

### Q2. “How accurate should peak traffic be?”

It does not need to be precise unless the interviewer provides a known ratio. State a reasonable assumption, such as 5× average, and use it consistently. The interview signal is that you recognize average load is insufficient and reserve headroom for bursts and failures.

### Q3. “What if my estimate is off by 2×?”

Usually nothing fundamental. Back-of-the-envelope math is designed to distinguish orders of magnitude. If 2× changes the design completely, that is a sign the design has no headroom. What matters is whether the estimate moves you from one machine to many, from database bytes to object-storage scale, or from ordinary HTTP traffic to massive media egress.

### Q4. “What should I write on the whiteboard?”

Keep only decision-driving numbers: peak read/write QPS, storage/day or/year if large, bandwidth if payloads are large, and concurrent connections if long-lived. Write the resulting design implication beside them.

## Common mistakes to avoid

- Calculating many values but never using them.
- Forgetting a peak/burst factor.
- Confusing requests/day with QPS.
- Treating every request as the same payload size.
- Ignoring hot keys or flash-event skew.
- Assuming “large DAU” automatically means every subsystem needs sharding.
- Spending more than a few minutes on arithmetic in a 45-minute interview.

## Short revision note

**Estimation recipe:** users → actions → average QPS → peak QPS → payload → bandwidth/storage → concurrency/skew → architecture consequence. Round aggressively and make assumptions explicit.

## Topics to revise

- [ ] QPS from daily volume
- [ ] peak vs average
- [ ] read/write ratio
- [ ] storage growth
- [ ] bandwidth
- [ ] concurrent connections
- [ ] traffic skew / hot keys
- [ ] turning a number into a design decision

## Interview-ready synthesis

### A strong 60–90 second explanation

I would estimate only the dimensions that affect the design. I start from user activity, convert to average and peak QPS, estimate payload/storage when large, and separately estimate concurrent connections for realtime systems. Then I state the architectural consequence of each important number. I round aggressively because the goal is order-of-magnitude reasoning, not capacity procurement.

### How this topic connects to the wider system

- Performance: QPS and payload reveal whether CPU, DB reads, bandwidth, or egress dominate.
- Scalability: peak and skew tell you whether horizontal capacity or hotspot mitigation is needed.
- Reliability: headroom must include bursts, retries, deployments, and failure of one capacity unit.
- Cost: storage retention and media bandwidth can dominate even when API QPS is modest.

### Revision flashcards with answers

**Why use ~10^5 seconds/day?**  
It makes mental conversion from daily operations to average QPS fast; 86,400 is close enough for interview estimates.

**What is the first number for chat?**  
Often both peak message rate and concurrent connections; sockets can dominate even with moderate message QPS.

**What is the first number for video?**  
Bandwidth/egress and concurrent viewers are often more important than metadata API QPS.

**What if estimate is uncertain?**  
State the assumption and design with headroom; explain which threshold would change the architecture.

**What is decorative estimation?**  
A calculation that never influences any decision; skip it.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Personal notes

```text
Concepts that are clear:

Concepts to revisit:

Three things to remember:
1.
2.
3.

Questions for later:
```

---

**Next:** Lesson 03 — Networking Fundamentals
