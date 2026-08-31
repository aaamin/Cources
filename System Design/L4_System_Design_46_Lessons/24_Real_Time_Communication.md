# Session 24 — Real-Time Communication

## Outcome

You should be able to choose between polling, long polling, Server-Sent Events, and WebSockets; explain persistent connection gateways, heartbeats, reconnection, presence, offline users, session routing, and backpressure for realtime systems.

## Start With the Product Requirement

“Realtime” is vague.

Ask:
- update latency target?
- one-way or bidirectional?
- update frequency?
- number of concurrent clients?
- offline behavior?
- ordering?
- can missed updates be fetched later?
- browser/mobile/server clients?

Choose the simplest communication model that satisfies these.

## Polling

Client periodically asks:

```text
every 5 seconds:
GET /updates?since=...
```

Advantages:
- simple;
- HTTP infrastructure;
- stateless request handling;
- easy retries.

Costs:
- wasted requests when no updates;
- latency up to polling interval;
- high request volume with many clients.

Good when:
- updates are infrequent;
- seconds/minutes of delay acceptable.

Do not reject polling automatically. It is often the simplest correct solution.

## Long Polling

Client sends request; server holds it until:
- update available;
- timeout.

Then client immediately opens another request.

Advantages:
- lower latency than fixed polling;
- works with HTTP request/response.

Costs:
- many open requests;
- reconnect overhead;
- harder scaling than simple polling.

Historically useful; WebSockets/SSE may now be cleaner for many realtime products.

## Server-Sent Events (SSE)

Persistent HTTP connection from server to client for one-way updates.

Good for:
- notifications;
- dashboards;
- live scores;
- server-driven status.

Advantages:
- simpler than full bidirectional WebSocket when client mostly receives;
- browser support/reconnect semantics.

Limitation:
- primarily server → client.

Client actions still use normal HTTP.

## WebSockets

Persistent bidirectional connection:

```text
Client ⇄ Connection Gateway
```

Good for:
- chat;
- multiplayer presence;
- collaborative interaction;
- bidirectional high-frequency events.

Costs:
- connection state;
- heartbeat/reconnect;
- routing;
- load balancing/draining;
- large connection capacity;
- backpressure.

Do not use WebSockets for a dashboard that updates once every five minutes unless a simpler model fails requirements.

## Persistent Connection Gateway

At scale, separate connection management from business processing.

```text
Clients
  ⇅
Connection Gateways
  |
Message/Presence services
  |
Storage / Event bus
```

Gateway owns live sockets.

Business service determines:
- destination users;
- message persistence;
- authorization.

## Connection Registry

Need to know where a user/device is connected.

Conceptually:

```text
user123 → gateway7 / connectionA
user123 → gateway12 / connectionB
```

Could be:
- distributed registry;
- partitioned routing service;
- pub/sub channel;
- gateway ownership computed by key.

Registry can be ephemeral.

Durable user/messages should not depend solely on connection registry.

## Multi-Device

One user may connect:
- phone;
- laptop;
- tablet.

Message routing may fan out to all devices.

Need per-device:
- connection;
- last delivered/read state;
- authentication;
- session revocation.

## Heartbeats

Connections can appear alive after network disappears.

Heartbeats/ping-pong:
- detect dead peers;
- refresh presence;
- release resources.

Too frequent:
- bandwidth/battery cost.

Too slow:
- stale presence/dead connections remain longer.

Presence does not need millisecond accuracy in most products.

## Reconnection

Mobile networks change.

Client should:
1. reconnect;
2. authenticate;
3. provide last seen sequence/cursor;
4. fetch missed durable events;
5. resume live stream.

Do not depend on socket delivery alone for durable messaging.

## Offline Users

For chat:

```text
send message
   ↓
persist message
   ↓
if recipient connected → push live
else → deliver on reconnect / push notification
```

Durable storage is separate from realtime transport.

WebSocket is not a database.

## Ordering

Network reconnect can reorder arrival.

Need explicit ordering semantics:
- per conversation sequence;
- event version;
- timestamp + tie breaker.

Do not say “WebSocket preserves all chat ordering globally.”

A single TCP connection preserves byte order, but:
- multiple senders;
- gateways;
- retries;
- reconnect;
- server processing

still require application ordering.

## Presence

Presence is typically soft/ephemeral state:
```text
online
last_seen
typing
```

Can tolerate eventual consistency.

Avoid durable DB write for every heartbeat at huge scale. Use ephemeral/distributed state and periodically persist meaningful state if required.

## Backpressure

A slow client cannot consume infinite updates.

Options:
- bounded send buffer;
- drop replaceable updates (e.g. latest stock price supersedes old);
- disconnect slow client;
- resume from cursor;
- prioritize messages.

For chat, durable history lets client catch up later.

For live cursor movement, dropping old positions may be fine.

## Connection Scaling

Estimate:
```text
concurrent connections
× memory/connection
× gateways
```

Also:
- file descriptors;
- network;
- heartbeat rate.

If 10M connections heartbeat every 30s:
```text
~333k heartbeat actions/sec
```
across fleet, even before messages.

This is why connection estimates matter.

## Sticky Routing

A live connection is inherently attached to one gateway while connected.

You may need:
- stable connection after handshake;
- registry for routing messages to correct gateway.

This is different from ordinary HTTP sticky sessions because the socket itself is stateful.

## Failure

Gateway dies:
- thousands/millions of clients disconnect;
- reconnect storm occurs;
- registry entries become stale;
- clients must fetch missed data.

Mitigations:
- randomized reconnect backoff;
- multiple gateways;
- capacity headroom;
- durable message history;
- TTL/heartbeat cleanup.

## Worked Example — Chat Message

Sender:
```text
Client A → Gateway A → Message Service
```

Message Service:
1. authenticate/authorize conversation;
2. assign/validate message ID/order;
3. persist durable message;
4. publish routing event.

Router:
```text
recipient online?
 yes → recipient gateway → socket
 no  → remain durable + optional push
```

Recipient acknowledgement/read receipt:
- separate event/state;
- idempotent.

On reconnect:
- client sends last known message sequence;
- server returns missing messages;
- resume live.

## Comparison Table

| Model | Direction | Connection | Best fit |
|---|---|---|---|
| Polling | client asks | short | infrequent updates |
| Long polling | server waits | repeated long request | near-realtime HTTP |
| SSE | server → client | persistent | one-way updates |
| WebSocket | bidirectional | persistent | chat/collaboration |

## Small Design Drills

1. Stock dashboard updates once/minute. Why might polling be enough?
2. Why isn't WebSocket message delivery alone durable?
3. What happens when a gateway with 100k sockets crashes?
4. Why is presence usually eventually consistent?
5. How should reconnect recover missed chat messages?
6. Why can old cursor-position updates be dropped but chat messages should not?

<details>
<summary>Answer key</summary>

1. Low update frequency and modest latency target make simpler polling efficient enough.
2. Connection can disappear; durable message must be stored independently.
3. Clients disconnect/reconnect; reconnect storm, routing registry cleanup, missed-data recovery.
4. Exact instantaneous online status is expensive and usually not business-critical.
5. Resume from last durable sequence/cursor and fetch missing history.
6. Cursor movement is replaceable ephemeral state; chat content is durable user data.

</details>

## Common Interview Mistakes

- Defaulting to WebSockets for every “realtime” prompt.
- Treating socket as durable storage.
- Ignoring reconnection.
- Ignoring multi-device.
- No backpressure for slow clients.
- Writing every heartbeat to relational DB.
- Claiming TCP order solves application ordering.
- Forgetting reconnect storms after gateway failure.
- No concurrent-connection estimate.

## Must Remember

- **Choose communication model from direction, frequency, and latency needs.**
- **Polling is often good enough.**
- **SSE is useful for server→client.**
- **WebSocket is bidirectional persistent transport, not storage.**
- **Realtime gateways hold connection state.**
- **Durable data must survive gateway failure.**
- **Reconnect from a durable cursor/sequence.**
- **Presence is usually soft/ephemeral.**
- **Slow clients need backpressure.**
- **Gateway failure creates reconnect storms.**

## Interview Revision Summary

Realtime checklist:

```text
Latency target?
One-way/bidirectional?
Polling/SSE/WebSocket?
Concurrent connections?
Gateway?
Connection registry?
Heartbeat?
Offline?
Multi-device?
Ordering?
Durable cursor?
Reconnect?
Slow client?
Gateway failure/reconnect storm?
```

## Explain Without Notes

Design realtime delivery for chat where users can have multiple devices, go offline, and reconnect after a gateway failure without losing messages.

## Completion Checklist

- [ ] I can choose polling/long polling/SSE/WebSocket.
- [ ] I understand connection gateways.
- [ ] I design reconnect/offline recovery.
- [ ] I separate presence from durable data.
- [ ] I reason about ordering/backpressure.
- [ ] I estimate connection-scale effects.
