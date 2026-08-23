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
