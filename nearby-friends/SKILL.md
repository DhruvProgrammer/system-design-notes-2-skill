---
name: nearby-friends-design
description: Design a nearby friends feature with real-time location updates; use when discussing WebSocket servers, Redis location cache, pub/sub fanout, and scaling live location sharing.
---

# Nearby Friends

## Purpose
Design a scalable backend for sharing user locations and discovering nearby friends, where locations change frequently and updates must be low-latency and eventually consistent.

## Key concepts
- **Difference from proximity service**: user locations change constantly; business addresses are relatively static.
- **Requirements**: see nearby friends with distance and timestamp, update nearby list every few seconds; low latency, reliability (occasional loss acceptable), eventual consistency.
- **Scope examples**: nearby radius (e.g., 5 miles, configurable), straight-line distance, user counts, location history, inactive friend timeout, GDPR assumptions.
- **Functional**: nearby friends list with distance/timestamp, frequent updates.
- **Non-functional**: low latency, reliability, eventual consistency.
- **Estimation**: location refresh interval (e.g., 30s), concurrent users, average friends, page size; compute location update QPS.
- **High-level design**: load balancer → REST API servers (friends, profiles) and stateful WebSocket servers (forward updates, seed nearby friends at init); Redis location cache with TTL; user DB (user/friendship); location history DB (e.g., Cassandra); Redis pub/sub for location update channels.
- **Periodic location update flow**: client → LB → WebSocket server → save to history DB, update cache and in-memory location, publish to user channel via Redis pub/sub → subscribed WebSocket servers receive, compute recipients, send updates.
- **API design**: WebSocket routines (periodic update, receive update, init with nearby friends, subscribe/unsubscribe friend); HTTP API for auxiliary tasks.
- **Data model**: location cache maps user_id to lat/long/timestamp (Redis with TTL); location history in relational/Cassandra table.
- **Scaling components**:
  - API servers: autoscaling.
  - WebSocket servers: scale out with graceful draining.
  - Client init: fetch friends, subscribe to channels, fetch locations, forward.
  - User DB: shard by user_id; possibly dedicated service.
  - Location cache: shard Redis; TTL limits memory; handle write load.
  - Redis pub/sub: pre-allocate channels to avoid dynamic creation; large memory for channels; many servers for push throughput; service discovery (ZooKeeper/etcd) for channel-to-server mapping; consistent hashing to minimize moves on scaling; scale cluster via daily job or overprovisioning; stateful handling and hand-off.
- **Adding/removing friends**: callback on client triggers subscribe/unsubscribe on the responsible WebSocket server.
- **Users with many friends**: cap friends (e.g., 5000); whale users may increase load on one server; enough servers needed.
- **Nearby random person**: geohash-based pub/sub channels; subscribe to one or more geohashes for bordering areas.
- **Alternative to Redis pub/sub**: Erlang for distributed processes and pub/sub; niche skill concern.

## Procedure
1. Define scope: nearby radius, distance model, user/friend counts, refresh interval, history, inactivity, compliance.
2. Estimate update QPS and concurrency.
3. Design high-level architecture with REST and stateful WebSocket servers, Redis location cache, user DB, location history, and Redis pub/sub.
4. Define WebSocket routines and HTTP APIs.
5. Design data models for location cache and history.
6. Scale each component, especially WebSocket servers and Redis pub/sub.
7. Use service discovery and consistent hashing for pub/sub channel distribution.
8. Handle friend add/remove, whale users, and optional random nearby person.
9. Consider alternatives like Erlang if appropriate.

## Tradeoffs and failure modes
- WebSocket servers are stateful and need careful scaling and draining.
- Redis pub/sub enables low-latency fanout but needs significant memory and many servers at high push rates.
- Eventual consistency is acceptable but means some delay and possible missed updates during scaling.
- Pre-allocating channels avoids dynamic creation overhead but uses memory.
- Alternative technologies (e.g., Erlang) may fit but have hiring/complexity tradeoffs.

## Checks
- Is real-time location update and fanout scalable?
- Are location cache and pub/sub scaled appropriately?
- Is service discovery and channel distribution robust?
- Are edge cases (whale users, friend changes, scaling) addressed?
