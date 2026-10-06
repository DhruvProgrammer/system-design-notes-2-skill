---
name: scaling-from-zero-to-millions
description: Scale a system from a single server to support millions of users; use when discussing horizontal scaling, load balancing, database replication, caching, CDNs, and sharding.
---

# Scale from Zero to Millions of Users

## Purpose
Understand the iterative journey from a single-server setup to a system that supports millions of users, including the core components and tradeoffs at each stage.

## Key concepts
- **Single server setup**: web app, database, and cache all run on one machine; requests arrive via DNS-resolved domain names.
- **Database separation**: move the database to a dedicated server to independently scale web and database tiers; choose SQL for structured data or NoSQL for unstructured/low-latency needs.
- **Vertical vs horizontal scaling**: vertical adds CPU/RAM to existing servers (limited, no redundancy); horizontal adds servers and requires a load balancer.
- **Load balancer**: distributes traffic across servers, provides redundancy when a server fails, and enables easy capacity addition.
- **Database replication**: master handles writes; slaves handle reads; improves performance and availability; plans needed for slave failure and master failover.
- **Caching**: store frequently read, infrequently modified data in memory; consider expiration, consistency, failure mitigation (multiple cache servers), and eviction (LRU common).
- **CDN**: caches static content (images, CSS, JS) on geographically distributed servers; consider cost, cache expiry, fallback, and invalidation.
- **Stateless web tier**: move session data to a shared datastore to enable horizontal scaling and auto-scaling.
- **Multi-data center**: geoDNS routing and data replication across centers for availability and latency; requires traffic redirection, data synchronization, and automated deployment.
- **Message queue**: durable, in-memory buffer for asynchronous communication; decouples producers and consumers.
- **Logging, metrics, automation**: track errors and health, gain performance/user insights, streamline testing/deployment/scaling.
- **Database scaling**: vertical scaling has physical/cost limits and SPOF risk; horizontal scaling (sharding) partitions data by a key (e.g., user_id); challenges include resharding, celebrity/hotspot problem, and cross-shard joins (often solved by denormalization).

## Procedure
1. Start with a single server and understand request flow: DNS → web server → HTML/JSON responses.
2. Separate the database onto its own server when web and database tiers need independent scaling.
3. Add a load balancer and multiple web servers for horizontal scaling and redundancy.
4. Add database replication: master for writes, slaves for reads; plan failover and recovery.
5. Introduce caching for frequently accessed, rarely changed data; define expiration and eviction policies.
6. Use a CDN for static content to reduce latency and origin load.
7. Make the web tier stateless by storing sessions in a shared datastore.
8. Deploy to multiple data centers with geoDNS and data replication for availability and latency.
9. Use message queues to decouple components and smooth asynchronous workloads.
10. Add logging, metrics, and automation for observability and operational velocity.
11. Scale the database with sharding when a single database can no longer handle the load; choose a shard key that distributes data evenly.

## Tradeoffs and failure modes
- Vertical scaling is simpler but hits hardware limits and lacks redundancy.
- Read replicas improve read performance but can serve stale data and require failover handling.
- Caching improves performance but introduces consistency challenges; cache failures should not become SPOF.
- CDNs improve static content latency but add cost and cache invalidation complexity.
- Sharding enables scale but complicates joins, resharding, and hotspot management; denormalization is a common workaround.

## Checks
- Is the web tier stateless?
- Is there redundancy at every tier?
- Are caching and CDNs used appropriately for the workload?
- Is the data tier scaled with sharding where needed?
- Are components decoupled with message queues and asynchronous processing where beneficial?
