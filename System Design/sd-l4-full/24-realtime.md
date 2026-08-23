# Lesson 24 — Real-Time Communication

**Phase:** Fundamentals  
**Session:** 24/46  
**Recommended time:** 60–90 minutes

## What you will learn

Learn polling, long polling, SSE, WebSockets, connection gateways, presence, heartbeats, and reconnect/catch-up behavior.

This is **study documentation**. Read the explanations first; use the review section only after you have studied the lesson.

## Polling

Clients periodically request updates. It is simple and robust but creates empty requests and latency up to the interval. It can be perfectly adequate for slow-changing status.

## Long polling

The server holds a request until data or timeout, then the client reconnects. It reduces empty polls but still cycles requests and is less natural for very frequent bidirectional messages.

## SSE

Server-Sent Events maintain a one-way server→client HTTP stream. They fit dashboards, notifications, or job progress where client writes still use ordinary HTTP.

## WebSockets

WebSockets maintain bidirectional connections and fit chat, collaborative apps, games, and presence. The operational cost is connection state: millions of open sockets, heartbeats, reconnects, gateway ownership, and routing.

## Connection gateways

Dedicated gateway servers can own client sockets while backend services remain mostly stateless. A routing/directory layer maps user/session → gateway so delivery events reach the right process.

## Presence and heartbeats

Presence is ephemeral and usually eventually consistent. Heartbeats extend a short TTL; expiry marks the user offline. Network partitions make instant perfect presence unrealistic.

## Reconnection

Clients disconnect frequently. Sequence numbers, offsets, or last-seen message IDs let reconnecting clients request missed events and deduplicate replays.

## Worked example — chat delivery

A and B hold WebSockets to gateways. A sends a message; the message service authenticates and persists it, then emits a delivery event to B's gateway. If B is offline, durable history remains; on reconnect, B fetches after its last acknowledged message. Presence is separate ephemeral state.

## Interview lens

Choose the simplest mechanism that fits update frequency and directionality. Do not use WebSockets simply because the system is “real-time.”

## What to remember

Real-time design is about connection lifetime, direction, routing, reconnects, ordering, and ephemeral state—not the protocol name alone.

## Review after reading

1. Weakness of polling?
2. Good SSE use?
3. Why are WebSocket servers stateful?
4. Heartbeat purpose?
5. How catch up after reconnect?

## Deeper study notes

### Connection count changes the server model

A stateless REST server may see each request independently. A WebSocket gateway owns a long-lived socket and associated routing state. Scaling involves connection distribution, heartbeat timers, memory per socket, load-balancer support, and reconnect storms after failure.

### Delivery is separate from connection

A WebSocket being online does not make it the source of truth for a message. Persist durable state first when the product requires no loss, then use the connection for low-latency delivery. If the connection fails, the client catches up from durable history.

### Reconnect storms are a real overload case

If one region or gateway tier restarts, millions of clients may reconnect simultaneously. Randomized reconnect backoff, admission control, and regional capacity prevent recovery traffic from causing a second outage.

### Presence is a lease

Think of online status as a short lease extended by heartbeats. It is expected to be stale briefly. Strongly coordinating every presence change globally would be expensive and unnecessary for most products.

### Common mistakes

- Assuming WebSocket messages are automatically durable.
- Keeping global exact presence in a strongly consistent database.
- Forgetting authentication/authorization when reconnecting.
- Requiring global message order.
- Ignoring reconnect storms and connection draining.


## Important interview ideas

> **Important:** Choose a real-time mechanism based on **direction, update frequency, connection count, and recovery semantics**. “Use WebSockets” is not the complete design.

### Polling

Polling is operationally simple and often good enough for updates every tens of seconds/minutes. Its cost is unnecessary requests and freshness bounded by poll interval.

For a job status page that changes once per minute, a 5-second poll may be acceptable. A WebSocket would add connection-management complexity for little benefit.

### Long polling

The server holds the request until an event arrives or timeout. It reduces empty requests compared with frequent polling but still creates repeated HTTP request lifecycles. Historically useful for chat-like systems where WebSockets were unavailable.

### SSE

Server-Sent Events use one long-lived HTTP response for server→client updates. Browser support and simple reconnection make SSE attractive for dashboards, notifications, and event feeds where the client sends commands separately over HTTP.

### WebSockets

WebSocket is bidirectional and persistent. Operational concerns include:

- millions of open sockets;
- heartbeat/idle timeout;
- gateway ownership;
- load-balancer draining;
- reconnect storms;
- authorization refresh;
- missed-message recovery.

The connection is ephemeral; important business events should also have durable state.

### Presence is a lease

Presence can be modeled as:

```text
user_id → gateway_id, last_heartbeat, expires_at
```

If heartbeats stop, presence expires. It is naturally approximate/eventually consistent. Do not put presence in the critical durable message path unless requirements demand it.

### Reconnect/catch-up protocol

A client sends its last acknowledged sequence/message ID:

```text
resume_after = msg_12345
```

Server sends missing messages from durable storage. This means live delivery is an optimization on top of durable history, not the only source of truth.

## Communication comparison

| Mechanism | Direction | Best fit | Main cost |
|---|---|---|---|
| polling | client→server checks | infrequent/simple updates | wasted requests/freshness |
| long polling | mostly server events | moderate realtime | request churn |
| SSE | server→client | dashboards/event feeds | one-way |
| WebSocket | bidirectional | chat/collaboration/games | stateful connection ops |

## Interview questions and model answers

### Q1. “Why not WebSockets for everything?”

They require persistent connection infrastructure, heartbeats, session routing, draining, and reconnect logic. If updates are infrequent and one-way, polling or SSE is simpler.

### Q2. “What if a WebSocket gateway dies?”

Clients detect disconnect, reconnect to another gateway, re-authenticate, and provide a resume cursor/last message ID. Durable message state is outside the gateway, so connection loss does not lose accepted messages.

### Q3. “How do you scale millions of sockets?”

Use horizontally scaled connection gateways partitioned by region/user, with a directory/routing layer mapping user→gateway. Size instances by connection memory/file descriptors and handle reconnect storms/draining.

### Q4. “How accurate is presence?”

Usually eventually consistent. Heartbeats and TTLs mean a user may appear online for a short time after disconnect. Strong global presence would be expensive and rarely necessary.

## Common mistakes to avoid

- WebSocket as durable storage.
- No reconnect/resume path.
- Exact global presence promise.
- Ignoring connection count in estimation.
- No heartbeat/draining behavior.
- SSE chosen for bidirectional interactive traffic without separate write path.

## Short revision note

Realtime design = **protocol choice + connection ownership + heartbeat/presence + reconnect + durable catch-up**.

## Topics to revise

- [ ] polling
- [ ] long polling
- [ ] SSE
- [ ] WebSockets
- [ ] gateway ownership
- [ ] heartbeat/TTL
- [ ] presence
- [ ] reconnect/resume

## Interview-ready synthesis

### A strong 60–90 second explanation

I select polling, long polling, SSE, or WebSocket based on update direction/frequency and connection cost. For persistent connections I design gateway ownership, heartbeat/presence, draining, reconnect, and durable catch-up so socket failure does not lose accepted business events.

### How this topic connects to the wider system

- Performance: persistent connections avoid repeated polling for frequent updates.
- Scalability: millions of sockets require connection-focused capacity planning.
- Reliability: reconnect cursor recovers missed events after gateway/network failure.
- Consistency: presence can be approximate while message history remains durable.

### Revision flashcards with answers

**SSE direction?**  
Server to client over a long-lived HTTP stream; client writes separately.

**WebSocket stateful?**  
It owns an active connection, so routing/draining/reconnect are operational concerns.

**Heartbeat?**  
Periodic liveness signal used to expire stale presence/session ownership.

**Resume cursor?**  
Last processed durable event/message position sent on reconnect.

**Polling good when?**  
Updates are infrequent and simplicity is more valuable than instant push.

### If the interviewer pushes deeper

Do not panic or jump to a named technology. Restate the new requirement, identify which assumption changed, and modify only the affected part of the design. A useful phrase is:

> “The original design optimized for ___. With this new requirement, the bottleneck/guarantee changes to ___, so I would introduce/change ___; the cost is ___.”

This is usually a stronger L4 signal than replacing the whole architecture.

## Cross-system connections

The value of this topic becomes clearer when you see it appear in different architectures:

- Chat: WebSocket bidirectional connection plus durable catch-up cursor.
- Job status: polling may be simpler because updates are infrequent.
- Live dashboard: SSE can stream server updates while commands remain normal HTTP.

### When not to overuse this idea

Do not use realtime push when the product tolerates seconds/minutes of delay and periodic polling is operationally simpler.

### A good interviewer sentence

> “I would use this only because the current requirement/workload creates the specific problem it solves. If that assumption changes, I would simplify or choose the alternative.”

This sentence captures an important L4 behavior: architecture is conditional, not dogmatic.

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

**Next:** Lesson 25 — Object Storage, Media, Multi-Region & Disaster Recovery
