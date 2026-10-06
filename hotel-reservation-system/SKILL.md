---
name: hotel-reservation-system-design
description: Design a hotel reservation system; use when discussing room-type reservation, inventory modeling, double-booking prevention, idempotency, locking, sharding, and service consistency.
---

# Hotel Reservation System

## Purpose
Design a reservation system for a hotel chain with many hotels and rooms, focusing on correct inventory handling, preventing double bookings, and scaling.

## Key concepts
- **Scope**: hotel chain with thousands of hotels and ~1M rooms; reserve room type, not specific room number.
- **Data model**: relational DB for clear relationships and ACID; inventory table per hotel, room type, date with total inventory and reservations; separate concerns for hotel info, rates, reservations, payments, management.
- **Correctness challenge**: prevent duplicate/oversold bookings.
- **Idempotency**: idempotency keys to handle repeated submissions by the same customer.
- **Concurrency control**: pessimistic locks, optimistic locking, DB constraints for concurrent bookings.
- **Overbooking**: possible policy (e.g., 10% overbooking).
- **Scaling**: inventory caching, read replicas, sharding; consistency across services.
- **Service consistency**: two-phase commit vs saga-style compensating transactions.

## Procedure
1. Define scope: hotels, rooms, reservation unit (room type), features, scale.
2. Model inventory by hotel, room type, date with total and reserved counts.
3. Use relational DB for ACID and clear relationships.
4. Add idempotency keys for repeat submissions.
5. Choose concurrency control: pessimistic lock, optimistic lock, or constraints.
6. Decide on overbooking policy if any.
7. Add caching for inventory, read replicas, and sharding as needed.
8. Handle cross-service consistency with 2PC or saga compensating transactions.

## Tradeoffs and failure modes
- Reserving room types simplifies inventory but requires careful availability tracking.
- RDBMS gives ACID but may need sharding/replicas at scale.
- Pessimistic locking reduces concurrency; optimistic locking can cause retries; constraints enforce correctness but need careful design.
- Overbooking can increase utilization but risks customer impact.
- 2PC is strong but can be slow and block; saga is more scalable but needs compensating logic.

## Checks
- Is inventory modeled correctly per hotel/room type/date?
- Are double bookings prevented via idempotency and locking/constraints?
- Is overbooking policy defined and enforced?
- Is cross-service consistency handled appropriately?
