---
name: back-of-the-envelope-estimation
description: Make quick rough calculations for system capacity and performance in design interviews; use when estimating QPS, storage, cache needs, and server counts.
---

# Back-of-the-Envelope Estimation

## Purpose
Perform fast, rough calculations to assess whether a design meets scale requirements, using explicit assumptions, labeled units, and common performance benchmarks.

## Key concepts
- **Powers of two**: understand data volumes in powers of two for accurate storage and bandwidth calculations.
- **Latency numbers**: relative performance of common operations (L1 cache ~0.5ns, L2 ~7ns, main memory ~100ns, SSD random read ~150µs, HDD random seek ~10ms, round-trip in data center ~500µs, inter-region ~150ms); memory is fast, disk is slow; avoid disk seeks; compress data before transmitting.
- **Availability nines**: 99% ≈ 3.65 days/year downtime, 99.9% ≈ 8.8 hours, 99.99% ≈ 52 minutes, 99.999% ≈ 5.3 minutes, 99.9999% ≈ 31.56 seconds; cloud SLAs often target 99.9% or higher.
- **Estimation targets**: QPS, peak QPS, storage requirements, cache requirements, number of servers.

## Procedure
1. State assumptions explicitly (active users, requests per user, media ratio, retention, etc.).
2. Estimate QPS from DAU and per-user activity; scale to peak (e.g., 2x average).
3. Estimate storage from data sizes and retention period.
4. Use round numbers and approximation; precision is not critical, the process matters.
5. Label all units to avoid ambiguity.
6. Apply powers-of-two and latency benchmarks to sanity-check sizes and performance.

## Tradeoffs and failure modes
- Over-precision adds little value; focus on a defensible process.
- Unstated or wrong assumptions can invalidate the entire estimate.
- Forgetting peak traffic can lead to underestimating capacity needs.
- Ignoring media/storage growth can cause large underestimates over multi-year horizons.

## Checks
- Are assumptions written down and labeled with units?
- Did you estimate both average and peak QPS?
- Did you account for storage growth over the required retention period?
- Are latency and availability numbers consistent with the design's goals?
