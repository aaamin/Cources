# Session 24 — Real-Time Communication

**Phase:** Phase 1 — Fundamentals  
**Recommended time:** 60–90 minutes

## Session Goal

Choose and operate the appropriate realtime communication model for updates, messaging, and presence.

## What You Need to Read / Learn

- Short polling and its simplicity/cost.
- Long polling.
- Server-Sent Events (server → client streaming over HTTP).
- WebSockets for bidirectional persistent communication.
- Connection gateways and connection/session routing.
- Heartbeats and dead-connection detection.
- Reconnect behavior.
- Presence and ephemeral state.
- Backpressure for slow clients.
- Delivery semantics separate from transport connection.
- Scaling millions of concurrent connections.

## What You Need to Do

- [ ] Choose polling/SSE/WebSocket for stock updates, notifications, chat, and job status.
- [ ] Design a connection gateway mapping user/device → active connection.
- [ ] Explain what happens to an outgoing chat message if the receiver is offline.

## **Must Remember for the Interview**

- **WebSocket is a transport choice; message durability, ordering, and delivery guarantees still require application design.**
- **Use persistent connections only when their benefits justify connection-management complexity.**
- **Presence is usually ephemeral and can tolerate weaker consistency than messages.**
- **Reconnects and duplicate delivery are normal cases.**
- **Slow clients need backpressure/buffering/disconnect policy.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Polling = simple but repetitive. SSE = server→client stream. WebSocket = bidirectional persistent connection.**
- **Connection gateways track live connections; durable messages live elsewhere.**
- **Separate online delivery from offline persistence.**
- **Define heartbeat/reconnect semantics.**
- **Do not claim WebSockets solve message ordering or exactly-once delivery.**

## Self-Test Before Marking This Session Complete

- [ ] Can I choose polling vs SSE vs WebSocket?
- [ ] Can I explain how connection routing works?
- [ ] Can I explain online vs offline message delivery?
- [ ] Can I identify slow-client/backpressure issues?

## Completion Rule

Mark this session complete only when you can explain the topic aloud, without notes, using **what it solves → how it works → when to use it → trade-offs → failure behavior → alternative**.


---

**Progress:** Session 24/46  
**Next:** Session 25
