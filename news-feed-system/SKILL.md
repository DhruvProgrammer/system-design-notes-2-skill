---
name: news-feed-system-design
description: Design a news feed system with publishing and retrieval; use when discussing fanout strategies, caching layers, and feed construction.
---

# Design a News Feed System

## Purpose
Design a scalable news feed system supporting publishing and reverse-chronological feed retrieval for many users and friends.

## Key concepts
- **Features**: publish posts, view friends' posts in reverse chronological order; support web and mobile; up to ~5,000 friends; ~10M DAU; text, images, video.
- **APIs**: POST /v1/me/feed to publish; GET /v1/me/feed to retrieve.
- **Feed publishing flow**: user → load balancer → web servers (auth) → post service (store post in DB/cache) → fanout service (propagate post ID to friends' feed caches) → notification service.
- **Feed retrieval flow**: user → load balancer → web servers → news feed service (fetch post IDs from feed cache, fetch post details from DB/cache).
- **Fanout strategies**:
  - **Fanout on write**: push posts to friends' feeds at write time; fast reads, expensive for users with many friends.
  - **Fanout on read**: pull posts at read time; efficient for inactive users, slower reads.
  - **Hybrid**: push for most users, pull for high-connection users (e.g., celebrities).
- **Fanout service**: fetch friend IDs from graph DB, filter via cache (mutes/selective sharing), send to message queue, fanout workers update feed cache with <post_id, user_id> mappings, append post IDs to friends' feed caches with a limit.
- **Cache layers**: news feed cache (post IDs), content cache (post details), social graph cache, action cache (likes/replies/shares), counter cache (counts).
- **Scaling**: database sharding and read replicas, stateless web tier.
- **Caching**: multiple layers to reduce latency and DB load.
- **Reliability**: consistent hashing, message queues.
- **Monitoring**: QPS, latency, cache hit rates.

## Procedure
1. Define requirements: platforms, features, sorting, scale, content types.
2. Define publishing and retrieval APIs.
3. Design high-level flows for publishing and retrieval.
4. Choose fanout strategy (write, read, or hybrid) based on follower distribution.
5. Implement fanout service with friend fetching, filtering, queueing, and cache updates.
6. Design cache layers for feeds, content, graph, actions, and counters.
7. Add database scaling, stateless web tier, and observability.

## Tradeoffs and failure modes
- Fanout on write gives fast reads but can be expensive for celebrities.
- Fanout on read reduces write cost but slows retrieval.
- Hybrid balances both but adds complexity.
- Caching improves performance but adds consistency and invalidation concerns.
- Message queues decouple fanout but add latency and operational needs.

## Checks
- Is fanout strategy appropriate for the follower distribution?
- Are feed and content caches designed with appropriate keys and limits?
- Is the system scalable and stateless where possible?
- Are monitoring and failure handling in place?
