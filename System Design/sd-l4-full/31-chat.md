# Lesson 31 — Design WhatsApp / Messenger

**Phase:** Guided Design  
**Session:** 31/46  
**Recommended time:** 90–120 minutes

## Why this system matters

This lesson is a **reference design**, not an architecture to memorize. Study how the requirements lead to the design. Then close the file and derive your own version.

## 1. Scope and requirements

- 1:1 text chat.
- Low-latency online delivery.
- Durable history and offline catch-up.
- Per-conversation ordering.
- Large groups as an extension.

## 2. Scale and workload shape

Estimate DAU, messages/user/day, peak message rate, bytes/message, history retention, and concurrent connections. Connection count often dominates gateway capacity even if users send few messages.

## 3. API / contract surface

Live connection protocol can include:

```text
SendMessage(client_message_id, conversation_id, body)
Ack(message_id)
```

History:

```http
GET /v1/conversations/{id}/messages?cursor=...
```


## 4. Data model

```text
Conversation(id, type)
Member(conversation_id, user_id)
Message(message_id, conversation_id, sender_id, seq?, created_at, body)
```

Partitioning by conversation gives locality/order but very large groups can hotspot.

## 5. High-level architecture

```text
Clients ↔ Connection Gateways
             ↓
        Message Service
         /           \
        v             v
 Message Store   Delivery/Event Bus
                      ↓
              Recipient Gateway
```

Persist the message before acknowledging accepted durability. Live delivery is an optimization over durable history, not the source of truth.

Walk through the main operation end to end. Be explicit about where durable state is written and what is synchronous versus asynchronous.

## 6. Deep dives

### Ordering

Global ordering is unnecessary. Preserve a stable order within a conversation using sequence numbers or a partition keyed by conversation.

### Offline delivery

Track a client's last acknowledged/seen message. On reconnect, fetch messages after that point. Presence only decides whether to attempt immediate delivery.

### Large groups

Naively writing one copy per member makes a 100k-person group expensive. Keep one canonical message and fan out delivery/events; use specialized partitions for giant groups.

## 7. Failure modes and recovery

- Gateway dies: clients reconnect; stale session-directory entries expire.
- Message stored but ACK lost: client retries same client_message_id; server dedupes.
- Delivery duplicates: receiver dedupes message_id.
- Presence stale: durable catch-up still guarantees eventual delivery.
- Hot group: isolate/split group traffic and avoid one-shard write bottleneck.
- Membership change race: authorization checked against correct membership version/state.

A design is incomplete until it has a failure story.

## 8. Trade-offs and evolution

Presence can be eventually consistent; accepted message durability and membership authorization need stronger guarantees. Partitioning by conversation simplifies ordering but creates hot-group risk.

## 9. How to present this in an interview

```text
Requirements
→ workload / scale
→ API + data model
→ simple HLD
→ main flows
→ one deep dive
→ failures
→ trade-offs
→ summary
```

Do not start by naming products. State the capability first.

## 10. Study exercise

After reading, close this file and redesign the system for 45 minutes. Change one assumption—10× scale, multi-region, stronger consistency, or a hot tenant—and adapt rather than reproducing the diagram.

## 11. Completion check

You understand the lesson when you can explain the workload shape, source of truth, main read/write flows, hardest problem, three failure scenarios, one alternative, and the central trade-off.

## More detailed walkthrough

### Message send flow

```text
1. Client sends client_message_id + conversation + body on WebSocket.
2. Gateway authenticates session and forwards to Message Service.
3. Service verifies membership.
4. Persist canonical message / assign server message_id or sequence.
5. Acknowledge sender after durability threshold.
6. Publish delivery event keyed by conversation/user.
7. Online recipient gateway pushes message.
8. Recipient ACK advances delivery/read state.
```

If the sender times out after step 4 and retries, the stable client message ID prevents creating a second message.

### Message ordering vs delivery ordering

Network delivery can arrive out of order even when canonical storage has an order. Clients can buffer briefly or display by sequence. Do not require one global sequence across all chats; it destroys parallelism for no product benefit.

### Delivery semantics

Useful states may be:

```text
accepted by server
stored durably
pushed to device
acknowledged by device
read by user
```

These are different facts. “Delivered” must be defined.

### Multi-device sync

A user can have phone + web + tablet. Treat each device as a consumer with its own last-seen position. A durable conversation log allows every device to catch up. Read receipts may be user-level or device-level depending on product semantics.

### Media messages

Do not send large media through the message event itself. Upload to object storage, then send a message containing media metadata/object reference. CDN handles download.

### Common interview mistakes

- Using WebSocket memory as durable message storage.
- Claiming exactly-once delivery to a phone.
- Ordering every message globally.
- Sharding by sender and making conversation history scatter across many shards.
- Ignoring reconnect and duplicate sends.

### Reusable patterns learned

Persistent connections + durable log, per-key ordering, idempotent client IDs, offline catch-up, presence as ephemeral state, and separating message metadata from media blobs.


## Detailed reference design

### Core requirements

Keep first version focused:

- 1:1 text messages;
- low-latency online delivery;
- durable history;
- offline catch-up;
- per-conversation ordering;
- multi-device as extension.

Do not include voice/video calls unless asked; they are a different media system.

### Workload shape

Messaging has **three different scales**:

1. message writes/reads;
2. durable history storage;
3. concurrent persistent connections.

A service with 50k messages/s may still have 10M WebSockets. Connection gateways and message storage scale differently.

### Connection layer

```text
Users ↔ Regional Connection Gateways
              ↓
       Session Directory
```

Gateway owns ephemeral socket state. Directory maps `user_id → gateway_id` with TTL/heartbeat. Durable user/message state lives elsewhere.

If a gateway dies, clients reconnect and directory entries expire. No accepted message should depend solely on gateway memory.

### Send flow

```text
Sender WS
  ↓
Gateway
  ↓ authenticate/route
Message Service
  ↓
validate membership
  ↓
allocate message_id / sequence
  ↓
persist durable message
  ↓
ack sender
  ↓
publish delivery event
  ↓
recipient gateway if online
```

Whether ack occurs before or after replication depends on durability guarantee.

### Offline flow

If recipient is offline, there is no need to hold an infinite per-user queue in memory. Message is durable in history. On reconnect, client sends last acknowledged message/sequence and fetches missing messages.

### Ordering

Avoid global order. The product usually needs order within one conversation.

Options:

- conversation-specific sequence assigned by owner/partition;
- monotonic distributed IDs that roughly preserve time plus per-conversation tie-break;
- event stream partitioned by `conversation_id`.

The exact approach depends on strictness. Network delivery may arrive out of order, so clients use sequence/message IDs to reorder/dedupe.

### Delivery semantics

Useful states:

```text
accepted by server
delivered to device
read by user
```

These are different. Acknowledgement events update derived status. Do not claim “delivered” just because persisted.

### Multi-device

A user may have phone + web + tablet. Session directory maps user to multiple gateways/device sessions. Each device tracks its own sync cursor. Read state may be per-user or per-device depending on product.

### Group chat

Small groups can fan out delivery to all online members. Very large groups can create hot partitions and enormous live fan-out.

For 100k-member groups:

- persist one canonical message;
- partition delivery work/batches;
- do not duplicate large message body 100k times;
- use membership snapshots/versioning;
- potentially let clients pull history rather than guarantee immediate push to every member.

### Media messages

Store media in object storage with signed upload/download URLs. Message record contains media reference/metadata. This keeps large bytes out of message DB and gateway paths.

## Failure walkthrough

### Sender retries after timeout

Use client-generated/stable message ID or idempotency key. If message was already stored, return existing result rather than creating duplicate.

### Delivery event duplicates

Recipient/device dedupes by message ID. Exactly-once socket delivery is not required for exactly-once visible message.

### Storage partition unavailable

Do not acknowledge durable acceptance until write meets required durability. The sender may see temporary send failure rather than losing an “accepted” message.

### Presence stale

A message may be routed to an old gateway and fail; delivery system falls back to durable/offline catch-up. Presence correctness is not message correctness.

## Interviewer follow-ups

### “How do you handle message deletion?”

Store tombstone/edit event/version. Propagate to devices asynchronously. Clarify whether delete-for-me or delete-for-everyone and time limits matter.

### “How do you partition?”

`conversation_id` gives history locality and ordering. Giant groups can hotspot; split large conversations into delivery/segment partitions while keeping logical conversation sequence.

### “How do read receipts scale?”

For 1:1, simple per-user read cursor. For huge groups, per-message per-member receipts explode; product may show aggregate/read-by subset or per-user last-read sequence rather than one row per message receipt.

## Common interview mistakes

- WebSocket as source of truth.
- Global message ordering.
- No offline catch-up cursor.
- Presence made strongly consistent globally.
- Ack before durable write despite durability requirement.
- One copy of message body per recipient.
- Large media through gateway/app server.

## Short revision note

**Chat pattern:** persistent connection gateway + durable conversation-partitioned message store + async delivery + reconnect cursor. Separate ephemeral presence from durable message correctness.

## Topics to revise

- [ ] connection gateways
- [ ] session directory
- [ ] send/ack flow
- [ ] per-conversation ordering
- [ ] offline catch-up
- [ ] delivery/read receipts
- [ ] idempotent send
- [ ] multi-device
- [ ] large groups/hotspots
- [ ] media object storage

## Interview-ready opening

> **Important:** Study the reasoning, not the exact diagram. A concise opening for this prompt could sound like this:

I’ll scope to 1:1 text messaging with durable history and realtime online delivery. I’ll estimate both message rate and concurrent connections, use connection gateways for sockets, persist before durable acknowledgement, and preserve ordering only within a conversation.

## How the design evolves at 10×

At 10× sockets, scale regional gateway fleet and session directory. At 10× messages, shard history by conversation. Giant groups need specialized split/fan-out strategy rather than simply adding more general shards.

## Quick revision flashcards

**Source of truth?**  
Durable message store; socket delivery is ephemeral.

**Ordering scope?**  
Per conversation, not global.

**Offline delivery?**  
Reconnect/fetch after last acknowledged sequence from durable history.

**Presence guarantee?**  
Usually eventually consistent via heartbeat/TTL.

## Two-minute closing template

At the end of practice, summarize in this order:

```text
1. source of truth / core architecture
2. most important scale or correctness decision
3. main failure-handling mechanism
4. central trade-off
5. first change at 10×
```

If you can close clearly without looking at notes, you probably understand the architecture rather than only recognizing it.

## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 32 — Design Twitter / Instagram News Feed
