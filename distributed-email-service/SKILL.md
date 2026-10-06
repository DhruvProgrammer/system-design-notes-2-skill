---
name: distributed-email-service-design
description: Design a large-scale email service; use when discussing SMTP/POP/IMAP, metadata storage, attachments, search, deliverability, and distributed mail processing.
---

# Distributed Email Service

## Purpose
Design a large-scale email system supporting sending, receiving, folders, attachments, and search with high availability and deliverability.

## Key concepts
- **Protocols**: SMTP for server-to-server delivery; POP and IMAP for clients; HTTP APIs for webmail.
- **Architecture**: web servers, real-time servers, metadata store, object storage for attachments, cache, search service, queues and workers for outbound/inbound mail; queues decouple processing and support retries when recipient servers are unavailable.
- **Metadata storage**: user-based partitioning; denormalized read/unread views; message IDs supporting time ordering; attachments stored separately.
- **Search**: distributed search engine (e.g., Elasticsearch) with async indexing updates.
- **Deliverability**: IP reputation, spam/virus checks, email authentication.
- **Availability/security**: replication, multi-DC, fault tolerance, privacy, security, compliance.

## Procedure
1. Define features: send, receive, folders, attachments, search, scale.
2. Choose protocols for server-to-server and client access.
3. Design architecture with web/real-time servers, metadata store, object storage, cache, search, and queues/workers.
4. Design metadata storage with partitioning, denormalized views, and message IDs.
5. Store attachments in object storage.
6. Add search service with async indexing.
7. Address deliverability: IP reputation, spam/virus checks, authentication.
8. Add replication, multi-DC, fault tolerance, privacy, security, compliance.

## Tradeoffs and failure modes
- Queues improve decoupling and retries but add latency and operational complexity.
- Metadata partitioning and denormalization improve scale and read patterns but add consistency and update complexity.
- Search indexing async is scalable but can be eventually consistent.
- Deliverability depends on reputation and filtering; misconfiguration can cause spam filtering or delivery issues.
- Multi-DC and replication improve availability but add consistency and operational needs.

## Checks
- Are protocols and access patterns supported appropriately?
- Is metadata storage designed for scale and read patterns?
- Is search integrated with async indexing?
- Are deliverability, security, and compliance addressed?
