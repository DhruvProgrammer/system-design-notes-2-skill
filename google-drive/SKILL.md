---
name: google-drive-design
description: Design a cloud file storage and sync service; use when discussing block-based storage, delta sync, conflict resolution, sharing, and notifications.
---

# Design Google Drive

## Purpose
Design a scalable cloud file storage and synchronization service with upload/download, cross-device sync, sharing, revision history, and notifications.

## Key concepts
- **Requirements**: upload/download files, sync across devices, file revisions, sharing with permissions, notifications on edits/deletes/shares; reliability (no data loss), fast sync, bandwidth efficiency, scalability (e.g., 10M DAU), high availability.
- **Constraints/examples**: free space, max file size, average upload size, upload frequency, total storage.
- **Single-server setup**: web server, metadata DB, storage directory organized by namespaces; inadequate at scale.
- **APIs**: simple and resumable uploads, download, list revisions.
- **Distributed improvements**: shard storage by user_id, use S3-like storage with cross-region replication, load balancer, metadata DB replication/sharding.
- **Sync conflicts**: when two users modify the same file, first processed wins; present both copies for user resolution.
- **Improved design**: user interaction via browser/app, block servers (split files into ~4MB blocks with hashes, stored in cloud storage), cloud storage, cold storage for inactive files, load balancer, API servers (auth, profile, metadata), metadata DB and cache, notification service (pub/sub for changes), offline backup queue.
- **Metadata DB schema**: user, file, block, file version tables.
- **File upload flow**: split/compress/encrypt blocks, upload to block servers/S3, metadata stored as pending, S3 callback updates status to uploaded, notification informs users.
- **File sync**: delta sync (only changed blocks), compression, conflict resolution (first wins, conflicting versions saved separately).
- **File download flow**: notification or poll triggers metadata fetch, then block download and reconstruction.
- **Notification service**: long polling for async notifications.
- **Storage optimization**: deduplication via hash-based block comparison, versioning limits, cold storage (e.g., Glacier).
- **Failure handling**: secondary load balancer, reassign block server tasks, promote DB slave, cross-region replication for storage, reconnect notification clients.

## Procedure
1. Define functional and non-functional requirements and constraints.
2. Start with a single-server model, then move to distributed storage with sharding and object storage.
3. Split files into blocks; store blocks in cloud storage; track blocks in metadata.
4. Design upload flow with block upload, metadata pending status, and completion callback.
5. Design sync with delta sync, compression, and conflict resolution.
6. Design download flow via notifications and block reconstruction.
7. Add notification service and offline queue.
8. Add deduplication, versioning limits, cold storage, and failure handling.

## Tradeoffs and failure modes
- Block-based storage enables dedup and delta sync but adds metadata and assembly complexity.
- Delta sync saves bandwidth but requires block-level change detection.
- Conflict resolution must decide between first-write-wins and user-mediated merge.
- Cold storage saves cost but increases retrieval latency.
- Notifications improve sync responsiveness but need reliable delivery and offline handling.

## Checks
- Are files split into blocks and stored efficiently?
- Is delta sync used to reduce bandwidth?
- Are conflicts handled and surfaced to users?
- Are notifications and offline sync supported?
- Are failure cases handled for each component?
