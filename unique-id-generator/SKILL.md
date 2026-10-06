---
name: unique-id-generator-design
description: Generate unique, time-sortable IDs in distributed systems; use when designing ID generation that must be unique, ordered, and high-throughput.
---

# Design a Unique ID Generator in Distributed Systems

## Purpose
Generate unique, date-sortable, 64-bit numerical IDs at high throughput (e.g., 10,000+ IDs/sec) without coordination bottlenecks.

## Key concepts
- **Requirements**: unique, numerical, fit in 64 bits, time-sortable (not strictly +1), high throughput.
- **Multi-master replication**: database auto_increment with step increments per server; hard to scale across data centers, IDs may not always increase with time, scaling issues when servers change.
- **UUID**: 128-bit identifiers generated independently; no coordination, easy scaling; exceeds 64 bits, not time-sortable, may be non-numeric.
- **Ticket server**: centralized database increments and assigns IDs; simple for small scale, numeric; SPOF, synchronization challenges.
- **Twitter Snowflake**: divide ID into sections:
  - Sign bit (1 bit): typically 0.
  - Timestamp (41 bits): milliseconds since custom epoch; ensures time ordering.
  - Datacenter ID (5 bits): up to 32 datacenters.
  - Machine ID (5 bits): up to 32 machines per datacenter.
  - Sequence number (12 bits): IDs per millisecond, up to 4096; resets each millisecond.
- **Considerations**: clock synchronization via NTP, tuning section lengths, high availability and fault tolerance.

## Procedure
1. Clarify ID requirements: uniqueness, sortability, bit size, throughput.
2. Evaluate options: multi-master, UUID, ticket server, Snowflake-style.
3. Choose a scheme that fits bit constraints and sortability needs.
4. For Snowflake-style, allocate bits for timestamp, datacenter, machine, and sequence.
5. Ensure clock synchronization (NTP) to reduce drift issues.
6. Plan for high availability: redundancy and failover for ID generators.

## Tradeoffs and failure modes
- Multi-master is simple but can produce non-monotonic IDs and scaling challenges.
- UUIDs are easy and uncoordinated but exceed 64 bits and are not time-sortable.
- Ticket servers are simple but create a SPOF.
- Snowflake-style supports scale and ordering but depends on clock sync and careful bit allocation.
- Clock skew can cause ID ordering issues; availability is critical for mission-critical generators.

## Checks
- Are IDs unique and within 64 bits?
- Are IDs time-sortable?
- Can the system meet throughput requirements?
- Is clock synchronization addressed?
- Is the generator designed for high availability?
