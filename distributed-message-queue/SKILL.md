---
name: distributed-message-queue-design
description: Design a distributed message queue with topics, partitions, replication, ordering, and delivery semantics; use when discussing brokers, consumers, WAL, batching, and exactly-once.
---

# Distributed Message Queue

## Purpose
Design a distributed message queue supporting producers and consumers, optional repeated consumption, ordering, retention, configurable delivery semantics, and high throughput with durability.

## Key concepts
- **Benefits**: decoupling, scalability, availability, performance.
- **Examples**: Kafka, RabbitMQ, RocketMQ, Pulsar, ActiveMQ, ZeroMQ; Kafka/Pulsar are event streaming platforms but converge in features.
- **Requirements examples**: text messages a few KB, repeated consumption, order preservation, ~2-week retention, many producers/consumers, configurable delivery (at-most-once, at-least-once, exactly-once), configurable throughput/latency.
- **Messaging models**:
  - **Point-to-point**: message consumed by exactly one consumer; removed on ack; no retention in traditional queues.
  - **Publish-subscribe**: messages to a topic; all subscribers receive all messages.
- **Topics, partitions, brokers**: split topic into partitions for scale; messages distributed across partitions; servers hosting partitions are brokers; FIFO within partition; offset is message position; partition key determines partition (e.g., hash(key) % numPartitions); consumer groups consume partitions.
- **Consumer groups**: set of consumers working together; messages replicated per group; each group has own offset; parallel reading improves throughput but hurts ordering; mitigate by one consumer per partition per group; consumers ≤ partitions.
- **High-level architecture**: clients (producer/consumer), brokers (hold partitions), data storage, state storage (consumer state), metadata storage (config/topic properties), coordination service (service discovery, leader election).
- **Data storage**: write-heavy, read-heavy, no update/delete (in this design), sequential access; WAL with segments; old segments read-only; efficient on HDD with sequential access and OS caching.
- **Message structure**: key (partition assignment), value (payload), topic, partition, offset, timestamp, size, CRC; key need not be unique.
- **Batching**: critical for performance; amortizes network cost; sequential writes and disk caching; tradeoff: larger batches = higher throughput, higher latency; smaller batches = lower latency, lower throughput.
- **Producer flow**: routing layer option vs embedded routing in producer; embedded reduces hops, enables batching, lets producer choose partition; batch size is throughput/latency tradeoff.
- **Consumer flow**: consumer specifies offset, receives chunk; push (low latency, can overwhelm) vs pull (consumer controls rate, better for batch, higher latency/no-message calls mitigated by long polling); pull is common.
- **Consumer rebalancing**: decides partition assignments on join/leave/partition change; coordinator broker per group (found by hashing group name); leader consumer generates dispatch plan; heartbeat-based rebalance.
- **State storage**: partition↔consumer mapping and offsets; frequent read/write, low volume, random, consistency important; e.g., ZooKeeper.
- **Metadata storage**: config and topic properties; small, infrequent changes, high consistency; e.g., ZooKeeper.
- **ZooKeeper**: hierarchical KV for config, synchronization, service discovery, leader election.
- **Replication**: partition replicated across brokers; one leader; producers write to leader; followers pull from leader; ack after enough replicas sync; replica distribution plan in ZooKeeper.
- **In-sync replicas (ISR)**: replicas in sync with leader; lag threshold; committed offset = messages synced by all ISR; ISR tradeoff: all ISR ack = durable but slower; leader-only ack = faster but less durable; ack=0 no wait.
- **Consumer reading**: connect to leader for partition; simple; limited connections per group; scale hot topics by more partitions/consumers; can read from ISR in some cases.
- **Scalability**:
  - Producer: easy add/remove.
  - Consumer: groups isolated; rebalance handles joins/leaves.
  - Broker failure: enough replicas; new leader elected; partition redistribution; min ISR balances latency/safety; replicas across brokers/DCs; mirroring for cross-DC.
  - New broker: temporary extra replicas until caught up, then remove unnecessary replica.
  - Partition add: notify producer, rebalance consumer; new partition for new messages.
  - Partition decrease: decommission, new messages to remaining partitions, old partition retained until retention expires, then truncate and free space; consumers read from all during transition.
- **Delivery semantics**:
  - **At-most-once**: async send, no retry; consumer commits offset immediately; possible loss.
  - **At-least-once**: ack=1/all, retry; consumer commits after processing; possible duplicates; good when dedup possible.
  - **Exactly-once**: costly.
- **Advanced features**:
  - **Message filtering**: separate topics vs tags vs payload filtering vs broker-side filtering; tradeoffs.
  - **Delayed/scheduled messages**: temporary storage, move to partition at right time; delay queues or hierarchical time wheel.

## Procedure
1. Define requirements: message size, retention, ordering, repeat consumption, delivery semantics, throughput/latency.
2. Choose messaging model (queue vs pub-sub) and partitioning strategy.
3. Design brokers, partitions, offsets, and consumer groups.
4. Design storage: WAL with segments, message structure, batching.
5. Design producer routing and batching.
6. Design consumer pull model, offset management, and rebalancing.
7. Use coordination service (e.g., ZooKeeper) for metadata, state, and leader election.
8. Implement replication and ISR with configurable ack.
9. Support scaling for producers, consumers, brokers, and partitions.
10. Implement delivery semantics and advanced features like filtering and delayed messages.

## Tradeoffs and failure modes
- Batching improves throughput but increases latency.
- Pull vs push tradeoff: control vs latency.
- Replication improves availability but adds latency and complexity; ISR configuration balances durability and speed.
- Ordering is per-partition; parallel consumption can break ordering unless constrained.
- Exactly-once is expensive; at-least-once with idempotency/dedup is common.
- Decommissioning partitions and rebalancing add operational complexity.

## Checks
- Are ordering, retention, and delivery semantics clearly defined?
- Is partitioning and consumer group design consistent with ordering needs?
- Are replication and ISR configured for the durability/latency tradeoff?
- Are rebalancing, failure recovery, and scaling handled?
