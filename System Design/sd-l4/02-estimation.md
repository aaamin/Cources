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
