---
name: metrics-monitoring-and-alerting-design
description: Design a metrics monitoring and alerting system; use when discussing collection (push/pull), time-series storage, aggregation, alerting, dashboards, and scale.
---

# Metrics Monitoring and Alerting System

## Purpose
Design a scalable metrics monitoring and alerting system for infrastructure metrics with low-latency queries, reliability, and flexibility for new technologies.

## Key concepts
- **Scope**: internal use; operational metrics (CPU, memory, disk, request rate); not logs or distributed tracing; alert channels (email, phone, PagerDuty, webhooks).
- **Scale examples**: large DAU, many server pools/machines/metrics; 1-year retention; downsampling policy (raw 7d, 1m resolution 30d, 1h resolution 1y).
- **Non-functional**: scalability, low query latency, reliability for critical alerts, flexibility.
- **Core components**: data collection, data transmission, data storage, alerting, visualization.
- **Data model**: time-series with metric name and optional tags/labels; line protocol format (e.g., Prometheus/OpenTSDB); series identified by name + labels + timestamp; efficient aggregation by labels; keep label cardinality low.
- **Storage**: specialized time-series DBs (InfluxDB, Prometheus, MetricsDB, Timestream, OpenTSDB/HBase); in-memory cache + on-disk storage; high write throughput; efficient aggregation; not a general-purpose DB.
- **High-level design**: metrics source → metrics collector → time-series DB → query service → visualization/alerting.
- **Metrics collection**:
  - **Pull**: collector fetches from /metrics endpoints; service discovery (ZooKeeper/etcd) for endpoints and schedule; easier debugging and health check; issues with short-lived jobs (use push gateway) and firewall/multi-DC reachability.
  - **Push**: agent pushes metrics; can aggregate before sending; good for short-lived jobs and firewalls; collector can be overloaded, so autoscale behind LB; any client can push (needs auth/whitelist); lower-latency transport (UDP) possible.
- **Transmission pipeline**: collectors in autoscale group; queue (e.g., Kafka) to buffer and decouple; stream processors (Flink/Storm/Spark) write to TSDB; Kafka partition per metric name for aggregation; can partition by tags/labels and prioritize.
- **Aggregation locations**: collection agent (simple), ingestion pipeline (stream processing, loses raw precision), query side (no loss, slower).
- **Query service**: decouples visualization/alerting from DB; cache layer possible; TSDB may be powerful enough; custom query languages (e.g., Flux) are common; SQL often ineffective for time-series.
- **Storage layer**: most queries are for recent data (e.g., past 26h); choose DB that exploits this; encoding/compression (e.g., delta encoding) reduces size; downsampling reduces disk usage.

## Procedure
1. Define scope: metrics types, scale, retention, downsampling, alert channels, out-of-scope items.
2. Define data model: metric name, labels, timestamps, line protocol.
3. Choose a time-series storage system appropriate for write-heavy, recent-data-skewed workloads.
4. Design collection: pull or push, service discovery or agents, autoscaling collectors.
5. Design transmission pipeline with buffering (e.g., Kafka) and stream processing if needed.
6. Decide where aggregation happens (agent, pipeline, query side).
7. Design query service with optional cache and integration to visualization/alerting.
8. Add alerting and visualization components.
9. Apply encoding, compression, and downsampling for storage efficiency.

## Tradeoffs and failure modes
- Pull vs push: debugging/health-check vs short-lived jobs/firewall/lower latency.
- Aggregation early reduces storage but loses precision; late aggregation preserves precision but can be slower.
- Kafka buffering improves reliability and decoupling but adds operational overhead; alternatives exist.
- High label cardinality can hurt storage and query performance.
- General-purpose DBs can work with expert tuning but specialized TSDBs are often better.

## Checks
- Are metrics, scale, and retention clearly defined?
- Is collection (push/pull) chosen for the environment?
- Is the storage system suited to time-series workloads?
- Is the pipeline resilient and scalable?
- Are alerting and visualization integrated effectively?
