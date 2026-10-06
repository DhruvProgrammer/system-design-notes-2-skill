---
name: key-value-store-design
description: Design a distributed key-value store with partitioning, replication, consistency, and failure handling; use when building or evaluating NoSQL stores.
---

# Design a Key-Value Store

## Purpose
Design a scalable, highly available distributed key-value store supporting put/get for small key-value pairs with automatic scaling, tunable consistency, and low latency.

## Key concepts
- **Single-server store**: hash table in memory; optimize with compression and spilling less-frequent data to disk; limited by memory.
- **CAP theorem**: consistency, availability, partition tolerance; only two of three can be guaranteed; partitions are inevitable in distributed systems, so CA is not realistic.
- **CP vs AP**: CP blocks writes on partition to preserve consistency; AP keeps accepting reads/writes, accepting stale data and reconciliation later.
- **Data partitioning**: consistent hashing distributes data across servers; supports automatic scaling and heterogeneity via virtual nodes proportional to server capacity.
- **Data replication**: replicate across N servers by walking the ring and picking the first N servers; place replicas in distinct data centers.
- **Consistency via quorum**: N replicas, W write quorum, R read quorum; W + R > N ensures strong consistency; W=1,R=N optimizes writes; R=1,W=N optimizes reads; W+R<=N weakens consistency.
- **Consistency models**: strong (latest write returned), weak (may see stale), eventual (converges over time).
- **Inconsistency resolution**: versioning with vector clocks; vector clock is [server, version] pairs; ancestor vs sibling detection; conflict resolution via application logic or client.
- **Failure detection**: gossip protocol with member IDs and heartbeat counters; at least two independent sources to mark a node down.
- **Temporary failures**: sloppy quorum uses healthy nodes for writes/reads; hinted handoff sends missed changes back on recovery.
- **Permanent failures**: Merkle trees for efficient synchronization; compare root hashes, then recurse to find inconsistent buckets; only sync differing data.
- **Data center outages**: replicate across multiple data centers.
- **Write path (Cassandra-like)**: commit log, memory cache, flush to SSTable when full.
- **Read path**: check memory cache; if absent, use Bloom filter to locate data in SSTables; retrieve and return.
- **Final architecture**: simple get/put APIs, coordinator node as proxy, ring-based decentralized nodes, replication, no SPOF.

## Procedure
1. Define requirements: small key-value pairs, high availability, scalability, automatic scaling, tunable consistency, low latency.
2. Choose a distributed design that addresses CAP tradeoffs.
3. Partition data with consistent hashing and virtual nodes.
4. Replicate across N servers on the ring, spreading across data centers.
5. Configure quorum settings (N, W, R) to balance latency and consistency.
6. Add versioning and vector clocks for conflict detection and resolution.
7. Implement failure detection with gossip and heartbeat monitoring.
8. Handle temporary failures with sloppy quorum and hinted handoff.
9. Handle permanent failures with Merkle tree synchronization.
10. Design write and read paths with commit log, memory cache, SSTables, and Bloom filters.

## Tradeoffs and failure modes
- Strong consistency requires quorum coordination and can increase latency.
- Vector clocks add client complexity and can grow large; trimming may be needed.
- Sloppy quorum improves availability but can temporarily serve stale data.
- Merkle trees reduce sync traffic but add computation.
- Decentralized ring designs simplify scaling but require careful failure handling.

## Checks
- Is data partitioned and replicated appropriately?
- Do quorum settings match consistency and latency goals?
- Are conflicts detected and resolved?
- Are temporary and permanent failures handled?
- Do write and read paths use caching and indexing efficiently?
