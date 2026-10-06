---
name: ad-click-event-aggregation-design
description: Design near-real-time ad click aggregation; use when discussing streaming/batching, Kafka, windowing, watermarks, deduplication, and reconciliation.
---

# Ad Click Event Aggregation

## Purpose
Design a near-real-time system to aggregate ad clicks at high volume, supporting per-ad counts, top-clicked ads, filters, and reconciliation for billing accuracy.

## Key concepts
- **Requirements**: count clicks per ad over time ranges, compute most-clicked ads on a schedule, support filters (user, IP, country), near-real-time.
- **Design approach**: Kafka to decouple producers from processing; MapReduce-style aggregation pipeline; Cassandra for raw and aggregated data; raw events retained for debugging/backfills; pre-aggregated results for dashboard speed.
- **Stream processing concerns**: event time and watermarks for late events; tumbling and sliding windows; duplicate handling and exactly-once processing; recovery using offsets and snapshots.
- **Scaling**: consumers and aggregation workers; handling unusually popular ads; monitoring lag and health.
- **Reconciliation**: daily batch reconciliation against raw data for billing accuracy.
- **Alternatives**: Hive, Elasticsearch, ClickHouse, Druid.

## Procedure
1. Define requirements: counts, top ads, filters, latency, retention, accuracy.
2. Use Kafka to ingest click events and decouple producers from processing.
3. Design aggregation pipeline: raw event storage, aggregation workers, aggregated results.
4. Handle event time, watermarks, and windowing (tumbling/sliding).
5. Implement deduplication and exactly-once semantics where needed.
6. Design recovery via offsets and snapshots.
7. Scale consumers/workers and handle hot ads.
8. Add monitoring for lag and health.
9. Reconcile aggregated results with raw data periodically.

## Tradeoffs and failure modes
- Streaming gives near-real-time but adds complexity for late events and exactly-once.
- Batching is simpler but less real-time.
- Raw event retention supports debugging and backfills but adds storage.
- Aggregation improves dashboard performance but can lose detail if not backed by raw data.
- Reconciliation catches discrepancies but is periodic and may not prevent all billing issues in real time.

## Checks
- Are counts and top ads computed within latency goals?
- Are late events and duplicates handled?
- Is recovery and offset management in place?
- Is reconciliation performed for billing accuracy?
