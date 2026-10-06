---
name: rate-limiter-design
description: Design an API rate limiter; use when choosing throttling rules, algorithms, placement, or distributed enforcement.
---

# Design a Rate Limiter

## Purpose
Control request rates to protect services, manage costs, and prevent abuse while keeping latency and memory use low.

## Key concepts
- **Why rate limit**: prevent DoS/resource starvation, reduce costs, prevent overloads.
- **Placement options**: client-side (unreliable), server-side (preferred for control), or middleware/API gateway (flexible, good for microservices).
- **Algorithms**:
  - **Token bucket**: tokens added at a fixed rate; each request consumes a token; supports bursts; needs parameter tuning.
  - **Leaking bucket**: fixed-rate processing via FIFO queue; stable outflow; bursts may delay recent requests.
  - **Fixed window counter**: time divided into fixed intervals with counters; simple; bursts at window edges can exceed limits.
  - **Sliding window log**: track timestamps for a rolling window; accurate; higher memory use.
  - **Sliding window counter**: combines fixed window and sliding log for smoothing; memory-efficient approximation.
- **Storage**: use in-memory caching (e.g., Redis) for fast counter operations.
- **Distributed concerns**: race conditions and synchronization; use locks, Lua scripts, or sorted sets; centralized stores for synchronization; eventual consistency may be acceptable.
- **Monitoring**: track analytics to validate algorithm effectiveness and adjust rules.

## Procedure
1. Clarify requirements: server-side vs middleware, throttle rules, distributed needs, standalone vs in-app, user feedback on throttling.
2. Define requirements: accurate throttling, minimal latency, low memory, distributed capability, clear exceptions, high fault tolerance.
3. Choose placement based on current stack and whether an API gateway is in use.
4. Select an algorithm based on business needs (burst tolerance, accuracy, memory).
5. Implement counters in fast shared storage; check counters and allow/reject requests.
6. Address distributed synchronization with atomic operations or locks.
7. Add monitoring and alerting on throttling behavior.

## Tradeoffs and failure modes
- Token bucket is simple and burst-friendly but requires careful tuning.
- Leaking bucket smooths output but can delay recent requests under burst.
- Fixed windows are simple but can allow edge bursts.
- Sliding logs are accurate but memory-heavy.
- Distributed enforcement adds latency and consistency complexity; decide fail-open vs fail-closed behavior.

## Checks
- Are throttle rules defined per identity and time period?
- Is the algorithm matched to burst and accuracy needs?
- Are counter operations atomic in a distributed setting?
- Is throttling observable and documented?
