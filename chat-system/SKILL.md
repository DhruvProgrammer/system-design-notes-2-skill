---
name: chat-system-design
description: Design a real-time chat system with one-on-one and group chat, presence, multi-device sync, and push notifications; use when discussing WebSockets, message storage, and presence.
---

# Design a Chat System

## Purpose
Design a real-time chat system supporting one-on-one and group chat, online presence, multiple devices, push notifications, and permanent history at large scale.

## Key concepts
- **Requirements**: one-on-one and group chat (e.g., max 100), text messages (e.g., up to 100k chars), online/offline indicators, multiple devices, push notifications; ~50M DAU; permanent history.
- **Protocols**:
  - **HTTP**: sending messages with persistent connections.
  - **Polling**: inefficient, frequent redundant requests.
  - **Long polling**: keeps connection open until messages arrive; inefficient for inactive users.
  - **WebSocket**: bidirectional persistent connection for real-time send/receive; preferred.
- **Components**:
  - **Stateless services**: signup, login, profile; integrated with service discovery.
  - **Stateful services**: chat servers maintain persistent WebSocket connections; message delivery and synchronization.
  - **Third-party integration**: push notification services.
  - **Database**: key-value store for chat history (horizontal scaling, low latency, good for long tail; used by Facebook Messenger, Discord).
- **Data models**: one-to-one (message id as primary key), group chat (composite key channel_id, message_id); IDs via global 64-bit sequence (e.g., Snowflake) or local sequence within a channel.
- **Service discovery**: recommend best chat server by location/capacity (e.g., Apache ZooKeeper).
- **One-on-one flow**: User A → Chat Server 1 → assign ID, store → forward to Chat Server 2 if User B online, else push notification.
- **Group chat**: copy messages to individual inboxes; simplifies sync but can be expensive for large groups; recipient inbox (message sync queue) collects messages from multiple senders.
- **Message synchronization**: each device tracks cur_max_message_id; new messages have recipient ID = current user and message ID > cur_max_message_id.
- **Online presence**: heartbeat mechanism (periodic heartbeats; missing beyond threshold → offline); fanout via pub-sub per friend pair channel; effective for small groups.
- **Scalability**: horizontal scaling, load balancing, caching.
- **Error handling**: retries, queuing, service discovery for new servers on failure.
- **Extensions**: media support with compression/storage, end-to-end encryption, client-side caching, geo-distributed caching.

## Procedure
1. Define features, scale, and storage requirements.
2. Choose WebSocket for real-time bidirectional communication.
3. Separate stateless account services from stateful chat servers.
4. Use a key-value store for chat history with appropriate key design.
5. Implement service discovery to route clients to appropriate chat servers.
6. Implement one-on-one and group messaging flows with ID assignment and storage.
7. Implement message synchronization across devices via cur_max_message_id.
8. Implement presence with heartbeats and pub-sub fanout.
9. Add scalability, caching, retries, and extensions as needed.

## Tradeoffs and failure modes
- WebSockets give real-time performance but require stateful servers and connection management.
- Group message copying simplifies sync but can be expensive at scale.
- Presence fanout via pub-sub works well for small groups but needs scaling for many friendships.
- Key-value stores suit chat history but require careful key design and scaling.
- Multi-device sync requires consistent message ID ordering and device state tracking.

## Checks
- Are real-time and presence requirements met?
- Is message ordering and sync handled correctly across devices?
- Is the storage choice appropriate for scale and latency?
- Are presence and group messaging scaled appropriately?
