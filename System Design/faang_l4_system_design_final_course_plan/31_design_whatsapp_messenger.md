# Session 31 — Design WhatsApp / Messenger

**Phase:** Phase 2 — Guided System Design  
**Recommended time:** 90–120 minutes

## Session Goal

Combine realtime connections, durable messaging, offline delivery, ordering, presence, and multi-device behavior.

## What You Need to Read / Learn

- Before the attempt, quickly review the relevant fundamentals; do not study a full reference architecture.
- Scope: 1:1 first, then groups; text first; online/offline delivery; message history.
- Persistent connections through connection gateways.
- Routing user/device to connection gateway.
- Message IDs and deduplication.
- Durable message storage partitioned by conversation or another defended key.
- Per-conversation ordering versus global ordering.
- Acknowledgements/delivery status.
- Offline delivery and multi-device sync.
- Presence as ephemeral data with TTL/heartbeats.
- Large groups and fan-out trade-offs.
- After your first design, compare against trusted reference material and note only the highest-impact omissions.

## What You Need to Do

- [ ] Design 1:1 messaging first and narrate sender → storage → receiver flow.
- [ ] Change request: support 100k-member groups.
- [ ] Explain how reconnecting clients avoid duplicate/missing messages.

## **Must Remember for the Interview**

- **WebSockets manage connections; they do not provide durable message storage or exactly-once delivery.**
- **Define ordering scope precisely—usually per conversation, not global.**
- **Use stable message IDs and idempotent processing to survive retries/reconnects.**
- **Presence can be eventually consistent/ephemeral; message history cannot simply disappear.**
- **Large-group fan-out changes the architecture and may require asynchronous distribution.**

## **Interview Revision Summary**

> Re-read this section during final interview revision.

- **Connection gateway → message service → durable store/queue → recipient gateway.**
- **Offline recipients read from durable state later.**
- **Partition for conversation access + ordering while watching hot groups.**
- **Delivery status is a state machine, not magic exactly-once transport.**
- **Presence uses heartbeat/TTL and weaker consistency.**

## Self-Test Before Marking This Session Complete

- [ ] Did I distinguish connection state from durable message state?
- [ ] Did I define per-conversation ordering?
- [ ] Did I handle reconnect/duplicates?
- [ ] Did I handle offline users?
- [ ] Did I address hot/large groups?

## Completion Rule

Mark this session complete only after a first attempt, rubric score, review, and a targeted redo of the weakest section. **Do not memorize a reference diagram.**


## Session-Specific Notes

**Required change request:** Support very large group chats.

**Score using the 40-point rubric.**


---

**Progress:** Session 31/46  
**Next:** Session 32
