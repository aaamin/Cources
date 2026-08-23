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
          /          Message Store   Delivery/Event Bus
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


## Personal notes

```text
Patterns learned:

Mistakes I would likely make:

One decision to remember:

Questions to revisit:
```

---

**Next:** Lesson 32 — Design Twitter / Instagram News Feed
