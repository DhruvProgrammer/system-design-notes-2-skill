---
name: real-time-gaming-leaderboard-design
description: Design a real-time gaming leaderboard; use when discussing Redis sorted sets, sharding strategies, rank queries, and DynamoDB alternatives.
---

# Real-time Gaming Leaderboard

## Purpose
Design a real-time leaderboard for a mobile game showing top players, a user's rank, and nearby ranks, with real-time score updates.

## Key concepts
- **Requirements**: top 10 display, user's specific rank, users around a user (bonus); real-time score updates reflected on leaderboard; scalability, availability, reliability.
- **Scale examples**: DAU/MAU, matches per day, peak QPS estimates for score updates and leaderboard fetches.
- **APIs**: POST /v1/scores (game servers only) to update score; GET /v1/scores for top 10; GET /v1/scores/{user_id} for user score/rank.
- **High-level architecture**: client → game service (validate win) → leaderboard service → leaderboard store; alternative direct client update is insecure; optional message queue if other services need results.
- **Data models**:
  - **Relational DB**: simple leaderboard table per month (or month column); insert/update score; top 10 via ORDER BY score DESC LIMIT 10 with index; rank via variable row numbering or subquery count; not performant for large scale or arbitrary rank queries.
  - **Redis sorted set**: hash map (user→score) + skip list (score→users sorted); ZADD/ZINCRBY/ZRANGE/ZREVRANGE/ZRANK/ZREVRANK; O(logN) updates and lookups; good for top-K and rank; storage easily fits clustered Redis for millions of users.
- **Redis details**: monthly leaderboard keys; old months moved to historical storage; ZINCRBY for score updates; ZREVRANGE for top 10; ZREVRANK for rank; range query for nearby ranks; storage estimate ~hundreds of MB for all MAU; single Redis server can handle target QPS; replica for failover; persistence; MySQL for user details and event reconstruction; cache top-10 user details.
- **Serverless/Cloud option**: manage own Redis+MySQL vs AWS API Gateway + Lambda; Lambda is serverless, scales automatically; good for greenfield.
- **Scaling Redis**:
  - Range partitioning by score; maintain user→shard mapping (MySQL/cache); top-10 from highest-score shard; rank = within-shard rank + sum of higher-score shard counts (O(1) per shard counts).
  - Hash partitioning via Redis Cluster; scatter-gather for top-K; harder for rank; latency grows with partitions and K.
  - Fixed partitions preferred for this problem by the author; allocate extra memory for snapshots; benchmark with redis-benchmark.
- **NoSQL alternative (DynamoDB)**: write-heavy, sort within partition by score; use sort key for score; partition by month causes hotspot on latest month; write sharding by user_id % num_partitions; tradeoff partition count vs write vs read scatter-gather; rank is hard; percentile via periodic cron analysis.

## Procedure
1. Define features, scale, real-time needs, and API surface.
2. Choose storage: Redis sorted sets for low-latency top-K and rank; relational for smaller scale or supporting data.
3. Design score update and leaderboard fetch flows.
4. Use monthly leaderboard keys and archive old months.
5. Add MySQL for user details and recovery reconstruction.
6. Scale Redis via range or hash partitioning as needed; choose based on top-K and rank needs.
7. Consider DynamoDB with write sharding if preferred; accept rank limitations or use percentile approximation.
8. Add caching for top-10 user details and recovery paths.

## Tradeoffs and failure modes
- Redis sorted sets are excellent for top-K and rank but may need sharding at very high scale.
- Range partitioning makes top-K and rank easier; hash partitioning makes top-K scatter-gather and rank harder.
- DynamoDB scales well but rank queries are harder; write sharding reduces hotspots but increases read scatter-gather.
- Serverless reduces ops but may add latency/cost considerations.
- Recovery depends on persistent event/source-of-truth storage.

## Checks
- Are top-K, rank, and nearby ranks supported efficiently?
- Is the storage choice appropriate for scale and latency?
- Is sharding chosen to balance write and read needs?
- Is recovery and user detail lookup handled?
