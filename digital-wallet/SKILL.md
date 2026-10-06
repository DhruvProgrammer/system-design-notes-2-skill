---
name: digital-wallet-design
description: Design a digital wallet for high-throughput balance transfers; use when discussing distributed transactions (2PC, TC/C, Saga), event sourcing, CQRS, Raft replication, and sharding.
---

# Digital Wallet

## Purpose
Design a digital wallet service for balance transfers between accounts at very high throughput with transactional correctness, reproducibility, and high availability.

## Key concepts
- **Scope**: transfers between digital wallets; high TPS (e.g., 1M TPS → ~2M TPS with two legs); strong correctness; reproducibility via replay, not just reconciliation; availability ~99.99%; no FX.
- **Estimation**: cloud relational DB ~1k TPS; 1M TPS would need ~1000 nodes for one leg, ~2000 for two legs; goal: increase per-node TPS to reduce node count.
- **API**: POST /v1/wallet/balance_transfer with from_account, to_account, amount (string), currency, transaction_id (idempotency key).
- **In-memory sharding (Redis)**: map<user_id, balance> in Redis; partition Redis cluster; Zookeeper for partition count and node addresses; stateless wallet service; scales but no atomic cross-partition transfers.
- **Distributed transactions — 2PC**: coordinator (wallet service) reads/writes multiple DBs; prepare phase, then commit/abort; downsides: lock contention, coordinator SPOF.
- **Distributed transactions — Try-Confirm/Cancel (TC/C)**: variant of 2PC with compensation; phase 1 try (reserve/change), phase 2 confirm or cancel; two independent transactions; parallelizable; example: try deduct A, NOP for C; confirm NOP for A, add C; cancel reverse.
- **TC/C failure modes**: coordinator must recover intermediate state via phase status tables (transaction ID/content, try status, second phase name, second phase status, out-of-order flag); brief inconsistent state is acceptable if recovered and funds not spendable; always deduct before add to prevent spending intermediary state; invalid options: NOP+A+; invalid A- + C+ without atomicity; out-of-order execution handled via out-of-order flag.
- **Saga**: sequence of independent operations across services; execute in order; on failure, roll back with compensating operations; choreography (events) vs orchestration (coordinator); orchestration preferred for wallet complexity; comparison with TC/C: TC/C parallelizable, Saga linear; both can see partial inconsistent state; both application-level.
- **Event sourcing**: command (intended action, can fail, non-deterministic, FIFO queue), event (fact, ordered, FIFO queue), state (what changed, e.g., account→balance), state machine (validates commands, applies events, deterministic, no external IO/randomness); reproducibility: replay events to reconstruct state; immutable event list + deterministic state machine guarantees replay success; audit questions answered: balance at time via replay, correctness via recalculation, logic correctness via running different code versions against events.
- **CQRS**: multiple read-only state machines query immutable events; supports balance queries without blocking writes.
- **High-performance event sourcing**: save commands/events to local disk (append-only fast on HDD) instead of external queue; cache recent commands/events in-memory; mmap for disk+in-memory caching; store state locally via SQLite/RocksDB (LSM for writes, cache for reads); periodic snapshots to avoid full replay; snapshots as large binary files in distributed storage (e.g., HDFS).
- **Reliable high-performance event sourcing**: state/snapshot regenerate from events; events are the reliability-critical data (commands are non-deterministic); replicate event log across nodes with Raft for no data loss and order preservation; leader active, followers passive; majority up keeps system running.
- **Distributed event sourcing**: single Raft group capacity-limited; shard into multiple Raft groups; implement distributed transactions via orchestrator (TC/C or Saga); CQRS read path can be slow with polling; reverse proxy to send commands and poll on behalf reduces load; push responses from read state machines to proxy for near-real-time; final lifecycle: Saga coordinator creates phase status, determines partitions, raft leader receives command, validates, converts to event, replicates, read state machine pushes response, coordinator proceeds to next operation, informs client.

## Procedure
1. Define scope, TPS, correctness, reproducibility, availability, and FX assumptions.
2. Estimate per-node TPS and node count tradeoff.
3. Start with Redis in-memory sharding for throughput; note lack of atomic cross-partition transfers.
4. Evaluate distributed transaction approaches: 2PC, TC/C, Saga; choose based on latency and parallelism needs.
5. Implement phase status tracking and out-of-order handling for TC/C.
6. Introduce event sourcing for reproducibility and audit; define commands, events, state, state machine.
7. Apply CQRS for read queries.
8. Optimize performance with local append-only storage, caching, mmap, RocksDB/SQLite, and snapshots.
9. Add Raft replication for event log reliability.
10. Scale to multiple Raft groups with distributed transaction orchestration and push-based read responses.

## Tradeoffs and failure modes
- 2PC is strong but can block and has SPOF.
- TC/C is parallelizable and compensable but has brief inconsistent states and needs careful recovery.
- Saga is flexible and orchestratable but linear and still has partial inconsistent states.
- Event sourcing enables replay and audit but requires deterministic state machine and reliable event log.
- Local storage optimizations improve performance but make service stateful; Raft adds reliability but adds coordination.
- Sharding across Raft groups enables scale but requires distributed transaction orchestration.

## Checks
- Is high TPS addressed via per-node optimization and sharding?
- Are distributed transaction semantics and recovery handled?
- Is event sourcing implemented for reproducibility and audit?
- Is the event log replicated reliably (e.g., Raft)?
- Are read queries served efficiently via CQRS?
