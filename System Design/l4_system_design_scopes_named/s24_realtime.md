# Session 24 — Real-Time Communication

## 1. Must Learn

### Polling
- **Understand:** Understand periodic client requests and when simplicity outweighs freshness/overhead.
- **Decision/trade-off:** Simple stateless infrastructure vs latency and repeated empty requests.

### Long polling
- **Understand:** Understand server holds request until update/timeout then client reconnects.
- **Decision/trade-off:** Fewer empty polls/lower latency vs connection/request management overhead.

### Server-Sent Events
- **Understand:** Know server→client streaming over a persistent HTTP connection.
- **Decision/trade-off:** Simple one-way streaming/reconnect support vs no native client→server stream.

### WebSockets
- **Understand:** Know long-lived bidirectional connection for interactive realtime traffic.
- **Decision/trade-off:** Low-latency bidirectional messaging vs connection-state/operational complexity.

### Persistent-connection infrastructure
- **Understand:** Understand connection gateways, heartbeats, reconnects, session routing, and connection state.
- **Decision/trade-off:** Realtime latency vs large connection counts/stateful routing.

### Presence & offline behavior
- **Understand:** Know presence is ephemeral/approximate and offline clients need durable catch-up.
- **Decision/trade-off:** Freshness vs correctness/complexity.

### Fan-out & backpressure
- **Understand:** Understand pushing to many connected clients and slow clients can overload gateways/buffers.
- **Decision/trade-off:** Realtime delivery vs capacity protection/drop/coalesce policies.

## 2. Should Know

- Reconnection should include resynchronization/catch-up, not assume no messages were missed.
- Connection gateways can separate long-lived connection handling from business services.
- Session routing matters if connection state is localized.

## 3. Recognition Only

- WebRTC
- MQTT
- HTTP streaming internals

## 4. Important Comparisons

- Polling vs long polling vs SSE vs WebSockets.
- One-way server push vs bidirectional communication.
- Stateless request handling vs stateful persistent connections.
- Presence state vs durable message/state storage.

## 5. Important Interview Questions

1. Do updates need server push, and how fresh must they be?
2. Is communication one-way or bidirectional?
3. How many concurrent connections must be supported?
4. What happens when a client disconnects and reconnects?
5. How are offline users caught up?
6. What happens when one client consumes updates too slowly?

## 6. Common Interview Mistakes

- **WebSockets by default** → Choose the simplest model that meets freshness/direction needs.
- **Treating presence as perfectly accurate** → Presence is usually soft/ephemeral.
- **No reconnect/catch-up logic** → Assume connections drop.
- **Persisting every heartbeat as durable business data** → Separate ephemeral connection state from durable state.
- **Ignoring slow clients** → Apply backpressure/drop/coalescing policy.

## 7. Communication

### Important Vocabulary

polling, long polling, Server-Sent Events, WebSocket, persistent connection, heartbeat, connection gateway, presence, reconnect, session routing, offline user, fan-out, backpressure

### Useful Interview Phrases

- “I’d choose the simplest communication model that satisfies the freshness and direction requirements.”
- “WebSockets are justified here because communication is frequent and bidirectional.”
- “After reconnect, the client needs a durable catch-up path rather than trusting the live connection alone.”

### Important Questions to Ask the Interviewer

- **Question:** “Is communication server→client only or bidirectional?”  
  **Why it matters:** Major protocol decision.
- **Question:** “What update latency is acceptable?”  
  **Why it matters:** Determines polling vs push.
- **Question:** “How many simultaneous connections?”  
  **Why it matters:** Determines gateway capacity/state design.

## 8. ⭐ Must Remember

1. Do not default to WebSockets.
2. Polling is often simplest when freshness requirements are loose.
3. SSE is strong for one-way server push.
4. WebSockets fit frequent bidirectional realtime interaction.
5. Persistent connections require heartbeat/reconnect/routing capacity.
6. Presence is usually ephemeral.
7. Offline recovery and slow-client backpressure matter.

## 9. Study Priority

1. Study first: polling, long polling, SSE, WebSocket comparison.
2. Study next: persistent connections, gateways, heartbeat/reconnect.
3. Finish with: presence, offline users, fan-out/backpressure.

## 10. Revision Checklist

- [ ] Choose a realtime transport from requirements.
- [ ] Explain persistent connection capacity.
- [ ] Handle disconnect/reconnect.
- [ ] Separate presence from durable data.
- [ ] Handle slow clients and fan-out.

---

**Scope rule:** Study to the depth needed to explain the choice, trade-off, scaling/failure behavior, and a reasonable alternative. Do not dive into implementation internals unless an interviewer explicitly asks.
