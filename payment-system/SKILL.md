---
name: payment-system-design
description: Design a payment system for e-commerce pay-in/pay-out; use when discussing PSP integration, double-entry ledger, idempotency, retries, reconciliation, and payment security.
---

# Payment System

## Purpose
Design a payment backend for e-commerce that handles pay-in and pay-out flows, reliability, reconciliation, and security, using a third-party PSP for card processing.

## Key concepts
- **Scope**: payment backend for e-commerce; supports credit cards and others via PSP; no direct card storage (PCI/compliance); global but one currency for interview; pay-in and pay-out; reconciliation; ~1M transactions/day (low TPS, so throughput is not the focus).
- **Actors**: payment service (coordinates, risk checks), payment executor (executes orders via PSP), PSP (moves money), card schemes, ledger (financial record), wallet (merchant balances).
- **Pay-in flow**: user places order → payment event to payment service → store event → payment executor for orders → executor stores order → executor calls PSP → on completion, payment service updates wallet → wallet stores balance → payment service calls ledger.
- **APIs**: POST /v1/payments with buyer info, checkout_id, card info, payment_orders; payment_order includes seller account, amount (string), currency, payment_order_id (idempotency key/nonnce forwarded to PSP), status; GET /v1/payments/{id} for status.
- **Data model**: payment_events (checkout_id PK, buyer/seller info, card info, is_payment_done), payment_orders (payment_order_id PK, buyer_account, amount, currency, checkout_id FK, status enum, ledger_updated, wallet_updated). Strong consistency over performance; SQL preferred for ACID, DBA market, tooling, financial track record.
- **Double-entry ledger**: every money movement affects two accounts (debit one, credit other); sum of entries is zero; end-to-end traceability.
- **Hosted payment page**: use PSP widget to avoid storing card data and reduce compliance burden; flow: checkout → payment service → PSP registration request (currency, amount, expiration, idempotency UUID) → PSP returns token → serve PSP-hosted page → user enters details → PSP processes → redirect back → async webhook updates payment result.
- **Pay-out flow**: similar to pay-in but money moves from e-commerce account to merchant; may use account payable provider (e.g., Tipalti); bookkeeping and regulatory needs.
- **Reconciliation**: nightly settlement file from PSP compared to internal state; detect internal inconsistencies (ledger vs wallet); mismatches handled manually by finance (classifiable standard, classifiable manual, unclassifiable investigation).
- **Handling delays**: high-risk manual review, 3D Secure; wait for webhook or poll; show pending status; provide check-in page or email update.
- **Communication patterns**: sync (HTTP) simple but poor at scale (long call chains, poor failure isolation, tight coupling, hard to buffer); async: single receiver (one consumer) vs multiple receivers (publish to all); pub-sub model fits payment side effects across services.
- **Failed payments**: track payment state; retry queue; dead-letter queue for terminal failures; inspect/debug.
- **Exactly-once**: at-least-once via retries + at-most-once via idempotency; retry strategies: immediate, fixed, incremental, exponential backoff (default), cancel; server can suggest Retry-After; idempotency via idempotency-key header (UUID) and unique constraint in DB; PSP-level nonce prevents duplicate PSP processing.
- **Consistency**: eventual consistency across services via exactly-once and reconciliation; replication lag can cause read inconsistencies; serve reads/writes from primary or use consensus (Paxos/Raft) or consensus DBs (YugabyteDB, CockroachDB).
- **Security**: HTTPS, encryption/integrity, SSL with certificate pinning, data replication/snapshots, DDoS rate limiting/firewall, tokens instead of card data, PCI compliance, fraud checks (AVS, CVV, behavior analysis).

## Procedure
1. Define scope, payment options, PSP usage, currency, pay-in/pay-out, reconciliation, scale.
2. Design pay-in and pay-out flows with payment service, executor, PSP, ledger, wallet.
3. Define APIs and data models with strong consistency and idempotency keys.
4. Implement double-entry ledger for traceability.
5. Integrate PSP via API or hosted payment page; use nonce for idempotency.
6. Add reconciliation against PSP settlement files and internal consistency checks.
7. Handle delays, retries, and failed payments with queues and dead-letter queues.
8. Implement idempotency and at-least-once with dedup.
9. Choose sync vs async communication appropriately.
10. Add security, PCI considerations, fraud checks, and consistency strategies.

## Tradeoffs and failure modes
- Hosted payment page reduces compliance burden but adds redirect/webhook flow.
- Sync communication is simple but doesn't scale well; async improves resilience and scaling but adds complexity.
- Exactly-once is expensive; at-least-once + idempotency is common.
- Reconciliation catches discrepancies but is periodic; real-time inconsistencies may still occur.
- Replication lag can cause read inconsistencies; consensus DBs or primary-only reads can mitigate.
- Security and PCI compliance constrain design choices (no raw card storage, tokens, encryption).

## Checks
- Are pay-in and pay-out flows clearly designed?
- Is idempotency implemented for payments and PSP calls?
- Is reconciliation in place for internal and external consistency?
- Are retries, dead-letter queues, and failure handling defined?
- Are security and PCI requirements addressed?
