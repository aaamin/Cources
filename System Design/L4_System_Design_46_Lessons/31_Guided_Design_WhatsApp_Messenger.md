# Session 31 — Guided Design — WhatsApp / Messenger

## Interview Prompt

> Design a WhatsApp/Messenger-like chat system supporting one-to-one messaging, online/offline users, message history, delivery/read status, and multiple devices.

Change request:
> Support very large group chats.

Spend **40–50 minutes** before continuing.

---

# STOP — Attempt First

Your design should explicitly address:
- WebSocket/connection management;
- APIs/events;
- message IDs;
- storage;
- per-conversation ordering;
- delivery semantics;
- offline sync;
- multi-device;
- group fan-out;
- hot groups;
- regional routing/failure.

---

# Reference Reasoning

## 1. Scope

Core:
- 1:1 messages;
- group chat later;
- near-realtime delivery;
- offline history;
- multi-device;
- delivery/read receipts.

Non-goals initially:
- voice/video calls;
- end-to-end encryption protocol internals;
- media transcoding;
- search.

Non-functional:
- low send/delivery latency;
- messages durable;
- very high availability;
- ordering defined per conversation;
- eventual presence/read receipt acceptable.

## 2. Estimates

Assume:
- 100M DAU;
- 20M concurrently connected;
- 50 messages/user/day.

Messages/day:
```text
5B/day ≈ 50k average msg/s
```

Peak perhaps 5×:
```text
~250k msg/s
```

Connections:
```text
20M sockets
```

This means connection gateway capacity is as important as message RPS.

## 3. Client APIs / Protocol

Connection:
```text
WebSocket / realtime channel
```

Send event:
```json
{
  "type":"send_message",
  "clientMessageId":"...",
  "conversationId":"...",
  "body":"..."
}
```

Server ack:
```text
accepted(messageId, sequence)
```

History:
```http
GET /conversations/{id}/messages?beforeSequence=...
```

Use cursor/sequence pagination.

## 4. Data Model

```text
Conversation
- id
- type
- created_at

Participant
- conversation_id
- user_id
- role
- joined_sequence

Message
- conversation_id
- sequence
- message_id
- sender_id
- body/object_ref
- created_at

DeviceState
- user_id
- device_id
- last_sync...
```

Primary access:
```text
messages by conversation ordered by sequence
```

A wide-column/partitioned store can fit at scale; SQL can work at smaller scale. Choose from requirements.

## 5. Ordering

Define:
> messages are ordered within a conversation according to server-assigned sequence.

Do not promise global order across all chats.

Possible sequence allocation:
- conversation partition leader/sequencer;
- monotonic local sequence;
- timestamp + tie-breaker with weaker semantics.

For giant hot groups, one strict sequencer can bottleneck. Requirement may permit server order from partitioned authority.

## 6. Message ID

Client sends `clientMessageId` for retry dedup.

Server assigns durable `messageId`.

Idempotency:
```text
UNIQUE(sender/device, clientMessageId)
```
or scoped mapping.

If client times out and retries, server returns same accepted message.

## 7. High-Level Architecture

```text
Clients
  ⇅
Connection Gateways
  |
Message Service / Conversation Router
  |
Partitioned Message Store
  |
Event/Delivery Bus
  |
Fan-out/Routing
  ↓
Recipient Gateways
```

Connection registry:
```text
user/device → gateway/connection
```

Presence store is ephemeral.

## 8. Send Flow

1. sender sends over socket;
2. gateway authenticates connection;
3. message service checks conversation membership;
4. dedupe by client message ID;
5. assign ordering/ID;
6. persist message durably;
7. acknowledge sender;
8. publish delivery event;
9. route to online recipient devices;
10. offline devices fetch later.

Persist-before-ack gives stronger durability.

## 9. Offline Users

Message remains in durable store.

On reconnect:
```text
client supplies last known sequence per conversation / sync cursor
```

Server returns missing messages/events.

Push notification can alert mobile user but is not the message source of truth.

## 10. Multi-Device

Each user has devices.

Routing:
```text
recipient user → all active device connections
```

State:
- delivered/read may be per device or user-level depending product;
- sent message should sync to sender's other devices.

Need device authentication and revocation.

## 11. Delivery Semantics

Statuses:
```text
accepted
delivered-to-device/server
read
```

Define semantics precisely.

Do not say “delivered” without meaning.

Receipts can be eventually consistent and batched.

## 12. Presence

Ephemeral:
```text
online
last seen
typing
```

Use distributed ephemeral store/gateway heartbeat.

Do not write heartbeat to durable DB every few seconds for 20M connections.

## 13. Partitioning

Messages:
```text
hash(conversation_id)
```
keeps conversation history/order local.

Problem:
giant group chat becomes hot partition.

Alternative:
- bucket by `(conversation_id, sequence_bucket)`;
- dedicated partition for huge group;
- split fan-out from storage;
- special-case “celebrity” groups.

## 14. Large Group Change Request

Suppose 1M-member group.

Naive per-message fan-out:
```text
1 message → 1M delivery jobs
```

Huge amplification.

Separate concerns:

### Message storage
Store once in group log.

### Online delivery
Publish group event through hierarchical fan-out/gateway subscriptions.

### Offline
Do not copy full message into 1M per-user inboxes if unnecessary.
On reopen, user reads group log from last sequence.

### Push notifications
Rate/notification preferences; likely not push every message for huge group.

Use hybrid:
- small groups: fan-out eagerly;
- huge groups: fan-out on read/online subscriber channels.

Similar to celebrity feed problem.

## 15. Hot Group

One group receives massive send/read traffic.

Mitigations:
- dedicated partition;
- replicated read copies/cache;
- hierarchical pub/sub;
- shard membership list;
- rate limit abuse;
- sequence authority sized separately.

Strict total order across millions of concurrent senders is expensive. Confirm whether per-group server order is required and acceptable to serialize writes for that group.

## 16. Gateway Failure

Gateway with 500k sockets dies:
- clients reconnect;
- reconnect storm;
- registry TTL cleans stale routes;
- clients resync durable history.

Mitigate:
- jittered reconnect;
- capacity headroom;
- multiple zones;
- connection distribution.

## 17. Message Store Failure

Need replication.
If partition leader fails:
- failover;
- recent uncommitted messages handling.

Only acknowledge after required durability.

## 18. Multi-Region

A conversation can have participants worldwide.

Simplest:
- each conversation has home region;
- writes route there;
- preserves ordering;
- replicated read/delivery events.

Cost:
- sender far from home region incurs latency.

Alternative multi-writer is much harder due ordering/conflicts.

For L4, home-region per conversation is a strong explainable design.

## 19. Security / Abuse

- authenticate devices;
- authorize membership;
- rate limit spam;
- block/report;
- media via signed object URLs;
- encryption at rest/in transit.
- End-to-end encryption is a separate deep topic if interviewer asks.

## Interview Questions

1. Why WebSockets?
2. How do offline users catch up?
3. How do you dedupe client retries?
4. What exactly is message ordering?
5. What happens if gateway crashes?
6. How would giant group change fan-out?
7. How do you avoid one giant group hot partition?
8. How does multi-region affect order/latency?
9. Is push notification the message delivery mechanism?

## Common Mistakes

- WebSocket as storage.
- Global message order.
- No client retry dedup.
- Heartbeats persisted to primary DB.
- One delivery row per member for million-member group without cost discussion.
- No reconnect/resync.
- Presence strongly consistent.
- “Kafka handles it” without partition/order key.
- No definition of delivered/read.

## Must Remember

- **Persistent connection and durable message storage are separate.**
- **Define order per conversation.**
- **Client message ID makes retries deduplicatable.**
- **Persist before durable acceptance acknowledgment.**
- **Offline sync uses durable cursor/sequence.**
- **Presence is soft state.**
- **Multi-device requires multiple connections and sync.**
- **Large groups need hybrid fan-out.**
- **Gateway failure must trigger safe reconnect/resync.**
- **Conversation home-region is a simple way to preserve order globally.**

## Self-Score

Use the 40-point rubric; if correctness/order or scaling scores below 3, redo the deep-dive section.
