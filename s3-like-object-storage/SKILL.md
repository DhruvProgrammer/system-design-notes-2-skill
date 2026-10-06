---
name: s3-like-object-storage-design
description: Design object storage like S3; use when discussing buckets, objects, metadata/data separation, replication, erasure coding, multipart uploads, versioning, and garbage collection.
---

# S3-like Object Storage

## Purpose
Design an object storage service similar to S3 with high durability, vast scale, low cost, flat object structure, and RESTful access.

## Key concepts
- **Storage types**: block (raw devices, high performance), file (folders/files on block), object (sacrifices performance for durability, scale, low cost; flat structure; no hierarchy; versioning; RESTful API; good for binary/unstructured/cold data).
- **Comparison**: block is mutable, higher cost, medium-high performance, strong consistency, medium scalability; file is mutable, medium-high cost/performance, strong consistency, high scalability; object is immutable (versioning), low cost, low-medium performance, strong consistency, vast scalability, RESTful access.
- **Terminology**: bucket (globally unique logical container), object (data + metadata in a bucket), versioning (multiple variants), URI (unique identifier), SLA.
- **Requirements examples**: bucket creation, upload/download, versioning, listing; small and large objects; 100PB/year; durability ~6 nines; availability ~4 nines; storage efficiency.
- **Estimation**: object size distribution, IOPS limits per disk, median sizes, object count, metadata size.
- **Design philosophy**: object immutability, key-value-like access via URI, write-once-read-many; separate metadata from data like inode vs file blocks; scale independently.
- **High-level design**: load balancer, API service (stateless orchestration, IAM), IAM (auth/authz/access control), data store (object ID-based), metadata store.
- **Upload**: create bucket via PUT; API verifies IAM, creates bucket entry; object PUT sends payload to data store, persists, returns UUID; API creates metadata entry.
- **Download**: GET with bucket/object path; API verifies IAM, retrieves UUID from metadata, fetches payload from data store.
- **Data store components**: data routing service (stateless, scales), placement service (virtual cluster map, heartbeats, consensus cluster e.g., Paxos/Raft), data nodes (store objects, replicate, heartbeats).
- **Data persistence flow**: API → data store → routing → primary data node → local save + replicate to two secondaries → response after replication; UUID-based replication group via consistent hashing; primary replicates before response (strong consistency over latency).
- **Data organization**: separate files per object wastes blocks/inodes at small-file scale; use WAL to merge small files into larger files; confine files to cores to reduce lock contention; object mapping table (object_id, filename, offset, size) in RocksDB or local SQLite per data node.
- **Durability**: replicate across failure domains; 3 copies can yield ~6 nines at typical HDD failure rates; erasure coding (e.g., 8+4) reduces storage overhead (~50% vs 200%) and can increase durability (~11 nines) at cost of access speed and computation; replication better for latency-sensitive, erasure coding for cost/durability.
- **Correctness verification**: checksums per file and per object; for erasure coding, verify each piece's checksum.
- **Metadata data model**: buckets (small, can fit one server, scale reads), objects (shard by hash(bucket_name, object_name) for name-based queries; listing by prefix is hard across shards; denormalized listing table sharded by bucket ID can help).
- **Versioning**: object_version as TIMEUUID; new version = new object_id; delete creates special object_id marking deletion; queries return 404.
- **Multipart upload**: initiate, get upload ID, upload chunks independently with etags (MD5), complete with upload_id + part numbers + etags; reassemble; garbage-collect old parts.
- **Garbage collection**: reclaim space from lazy deletion, orphan data, corrupted data; compaction copies non-deleted objects to new files, updates mapping via transaction, avoids many small files by compacting beyond threshold.

## Procedure
1. Define features, object size mix, scale, durability, availability, efficiency.
2. Choose object storage semantics: flat, immutable, versioning, RESTful.
3. Separate metadata and data services; design API, IAM, data store, metadata store.
4. Design upload/download flows with authorization, metadata, and data persistence.
5. Design data store: routing, placement, data nodes, replication group via consistent hashing.
6. Choose replication vs erasure coding for durability and cost.
7. Add checksums for integrity.
8. Design metadata schema and sharding; handle listing and versioning.
9. Add multipart upload for large files and garbage collection/compaction.

## Tradeoffs and failure modes
- Replication is simpler and lower-latency but costs more storage; erasure coding saves space and can improve durability but costs compute and access speed.
- Separating metadata and data improves scale but adds coordination.
- WAL merging helps small-file efficiency but serializes writes; core confinement reduces contention.
- Local metadata DB per node reduces network latency but is tied to the node.
- Multipart upload improves large-file upload reliability and parallelism but adds state and completion logic.
- Garbage collection/compaction is needed to reclaim space and avoid fragmentation.

## Checks
- Are durability and availability targets met (replication/erasure coding, failure domains)?
- Is metadata separated and scaled appropriately?
- Are small and large objects handled efficiently?
- Is versioning and multipart upload supported?
- Is garbage collection/compaction in place?
