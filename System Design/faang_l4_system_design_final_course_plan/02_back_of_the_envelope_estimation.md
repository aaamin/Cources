# Session 02 — Back-of-the-Envelope Estimation

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Become fast enough with rough scale calculations to use numbers as architecture inputs without wasting interview time.

## What You Need to Read / Learn

- DAU/MAU and requests per user: convert product assumptions into traffic.
- Average QPS versus peak QPS: understand why peak traffic, not just averages, drives capacity.
- Read/write ratio: recognize how it influences caching, replicas, and storage design.
- Payload size and bandwidth: estimate network pressure when objects are large or fan-out is high.
- Storage growth: writes/day × bytes/write × retention, with replication overhead when relevant.
- Concurrent connections: especially important for chat, realtime presence, and streaming.
- Traffic skew: celebrity users, hot keys, hot tenants, geographic concentration, and bursty events.
- Order-of-magnitude arithmetic: prefer 10^3/10^6 reasoning over fake precision.

## What You Need to Do

- [ ] Estimate QPS for 10M DAU with 20 reads and 2 writes per user/day.
- [ ] Estimate one year of storage for 100M messages/day at 1 KB/message.
- [ ] Estimate concurrent WebSocket connections if 5M users are online at peak.
- [ ] For every number, write one architectural consequence. Example: high read QPS → cache/read replicas may matter.

## **Must Remember for the Interview**

- **Estimation is useful only when it changes a design decision.**
- **Peak load and skew often matter more than average load.**
- **Use round numbers and state assumptions. Interviewers care about reasoning, not arithmetic theater.**
- **Separate request rate, storage rate, bandwidth, and concurrent connection problems.**
- **Always sanity-check the final order of magnitude.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **QPS ≈ requests/day ÷ 86,400; peak can be several times average.**
- **Storage ≈ writes/day × bytes/write × retention × replication factor.**
- **Bandwidth ≈ QPS × payload size.**
- **Realtime systems may be constrained by concurrent connections, not only QPS.**
- **Skew/hotspots can invalidate otherwise-good averages.**

## Self-Test Before Marking This Session Complete

- [ ] Can I estimate QPS in under 2 minutes?
- [ ] Can I explain when bandwidth matters more than request count?
- [ ] Can I give an example where an average hides a hotspot?
- [ ] Can I stop calculating once I have enough information to design?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 2/46  
**Next:** Session 3
