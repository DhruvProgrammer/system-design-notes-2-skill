---
name: stock-exchange-design
description: Design an electronic stock exchange; use when discussing order entry, risk checks, sequencing, matching engine, order book, market data, determinism, and low-latency optimization.
---

# Stock Exchange

## Purpose
Design an electronic stock exchange that efficiently matches buyers and sellers with low latency, determinism, fault tolerance, and risk checks.

## Key concepts
- **Scope**: stocks only; place/cancel limit orders; normal trading hours; real-time matches and order book; tens of thousands of concurrent users, ~100 symbols, billions of orders/day; risk checks (e.g., daily volume limits); sufficient funds check and withholding funds for pending orders.
- **Non-functional**: availability ~99.99%, fault tolerance and fast recovery, latency (ms round-trip, focus on 99th percentile), security (KYC, account management, DDoS protection).
- **Estimation**: 100 symbols, 1B orders/day, 6.5h trading; QPS ~43k, peak ~215k; higher volume at market open.
- **Business concepts**: broker mediates exchange and end users; institutional clients trade large volumes with specialized software (order splitting); limit vs market orders; bid (highest buy) vs ask (lowest sell); L1/L2/L3 quotes (best bid/ask, more levels, queued quantity per level); candlestick (open, close, high, low per interval); FIX protocol for securities transaction messaging.
- **High-level design**:
  - **Trading flow**: client → broker → client gateway (validate, rate limit, authenticate) → order manager (risk checks, wallet/funds check) → sequencer (sequence inbound orders and outbound fills for determinism, fast recovery/replay, exactly-once) → matching engine (maintain order book per symbol, match buy/sell, emit two fills, deterministic) → fills back through order manager/client gateway to brokers.
  - **Market data flow**: matching engine emits executions → market data publisher (build order book/candlesticks) → data service → brokers for timely data.
  - **Reporting flow**: reporter collects reporting fields (client_id, price, quantity, order_type, filled_quantity, remaining_quantity) → DB; not critical path; accuracy/compliance important.
- **Trading flow optimization**: critical path, highly optimized; matching engine is heart; sequencer is key for determinism.
- **Order manager**: manages order state, sends orders, receives fills; reduces bandwidth by passing only necessary info; state transition management; event sourcing is a viable solution.
- **Client gateway**: receives orders, sends to order manager; lightweight; multiple gateways (e.g., colo engine rented by broker in exchange DC).
- **Market data publisher**: receives executions, builds order book/candlesticks, sends to data service.
- **APIs**: RESTful for client gateway–broker; proprietary protocol for institutional low-latency; create order, get execution, get order book (L2), get candlesticks.
- **Data models**:
  - **Product/order/execution**: products describe symbol attributes; orders and executions processed in-memory on critical path, stored/recovered from sequencer, written by reporter, forwarded to market data.
  - **Order book**: list of buy/sell orders per symbol by price level; needs constant lookup by price level/volume, fast add/execute/cancel, best bid/ask query, iterate price levels; implementation: PriceLevel (price, totalVolume, orders), Book<Side> (side, limitMap price→PriceLevel), OrderBook (buyBook, sellBook, bestBid, bestOffer, orderMap); use doubly-linked list for O(1) add at tail, O(1) match/delete from head, O(1) cancel via orderMap with list references.
  - **Candlestick**: open/high/low/close/volume/timestamp/interval; CandlestickChart as linked list; optimizations: pre-allocated ring buffers, limit in-memory sticks and persist rest; in-memory columnar DB (e.g., KDB) for real-time analytics, persisted to historical DB after close.
- **Performance**: reduce tasks on critical path; shorten task time by reducing network/disk/exec time; modern exchanges often run on one big server; strip logging from critical path; use mmap as event store (e.g., file in /dev/shm for shared memory, no disk access); application loop pinned to one CPU to avoid context switching and lock contention.
- **Event sourcing**: store immutable state transitions; order manager receives new order event, validates, adds to internal state, sends to matching core; on match, OrderFilledEvent over mmap; components subscribe to event store; components hold copy of order manager packaged as library to avoid extra calls; sequencer is single writer sequencing events before forwarding to event store.
- **High availability**: 99.99% (8.64s/day downtime); backup instances on stand-by; automate failure detection and failover; stateless services horizontally scaled; stateful components process inbound but not outbound unless leader; heartbeats for leader detection within a server; extend with hot/warm server replica failover; replicate event store via reliable UDP.
- **Fault tolerance**: if warm instances also down (low probability), replicate core data across DCs in multiple cities; chaos engineering to find edge cases; manual failover initially until failure modes understood; leader election (e.g., Raft); backup frequency based on loss tolerance; for exchange, data loss unacceptable → frequent backup + Raft replication.
- **Matching algorithm (pseudo-code)**: handleOrder checks sequence, validates, creates order; handleNew matches against opposite book; handleCancel checks existence, removes, sets canceled; match iterates price level orders, matches min(leavesQuantity, order.quantity), removes, generates fill; FIFO within price level.
- **Determinism**: functional determinism via sequencer; latency determinism tracked via percentiles; GC events can cause spikes.
- **Market data publisher optimizations**: ring buffer for candlesticks (preallocated, lock-free), padding to avoid sequence number sharing cache line; limit granularity/price of info.
- **Distribution fairness/multicast**: ensure subscribers receive data simultaneously to prevent market manipulation; use multicast via reliable UDP; unicast/broadcast/multicast concepts; UDP unreliable, enhanced with retransmissions.
- **Colocation**: brokers colocate servers in exchange DC for lower latency; VIP service.
- **Network security**: isolate public from private services; caching for infrequently updated data; cacheable URLs; allowlist/blocklist; rate limiting for DDoS.

## Procedure
1. Define scope, order types, trading hours, scale, risk checks, funds checks.
2. Define non-functional requirements: availability, fault tolerance, latency, security.
3. Estimate QPS and peak.
4. Design high-level flows: trading, market data, reporting.
5. Design client gateway, order manager, sequencer, matching engine.
6. Design data models: products/orders/executions, order book (doubly-linked list + maps), candlesticks.
7. Optimize performance: single server, mmap event store, pinned application loop, minimal critical path.
8. Apply event sourcing and sequencer for determinism and recovery.
9. Design high availability and fault tolerance: backups, leader election, replication across DCs, Raft.
10. Implement matching algorithm and market data publisher optimizations (ring buffer, padding).
11. Address distribution fairness via multicast/reliable UDP and colocation.
12. Add network security measures.

## Tradeoffs and failure modes
- Running on one big server reduces latency but creates concentration risk; backups and replication mitigate.
- mmap/shared memory avoids disk access and context switching but is more complex and platform-specific.
- Determinism is critical for recovery and fairness; sequencer and event sourcing support it.
- Market data fairness requires multicast and careful distribution; UDP reliability must be handled.
- Colocation improves latency for some clients but is a privileged service.
- Security and DDoS protection are important for internet-facing components.

## Checks
- Is determinism ensured via sequencing and event sourcing?
- Is the matching engine and order book efficient and correct?
- Are latency, availability, and fault tolerance addressed?
- Is market data distributed fairly and securely?
- Are risk checks and funds verification integrated on the critical path?
