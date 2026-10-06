---
name: proximity-service-design
description: Design a nearby-business search service; use when discussing geospatial indexing (geohash, quadtree, S2), radius search, caching, and scaling read-heavy location queries.
---

# Proximity Service

## Purpose
Design a service that finds nearby businesses by location and radius, with low latency, high availability, and support for business CRUD and details.

## Key concepts
- **Requirements**: search businesses by lat/long and radius, business CRUD (not real-time), business details; low latency, privacy compliance, high availability; large DAU and business counts.
- **API design**: search nearby with lat, long, radius; business get/post/put/delete endpoints.
- **Data model**: relational DB for business details and geospatial index; primary-replica for read-heavy workload; some read/write inconsistency acceptable if business info is not real-time.
- **Two-dimensional search (naive)**: circle query with lat/long bounds; inefficient full scans; limited by one-dimensional indexes.
- **Geospatial indexing options**:
  - **Even grid**: fixed-size grids; uneven distribution problem.
  - **Geohash**: encodes lat/long into alphanumeric string; hierarchical grids; shared prefix implies proximity; boundary issues (nearby points on edges, close points across equator with no shared prefix); need to search neighboring grids.
  - **Quadtree**: recursively divides space into four quadrants; in-memory on each LBS server; built at startup; efficient for k-nearest; adapts to density; updates are complex.
  - **Google S2**: maps sphere to 1D index via Hilbert curve; good for geofencing and arbitrary areas; more complex.
- **Tradeoff comparison**:
  - Geohash: easy, fixed radius, easy updates; boundary issues, fixed grid size.
  - Quadtree: k-nearest, density-adaptive; more complex, rebalancing.
  - S2: advanced geofencing; harder to implement.
- **Scaling business table**: shard by business ID for even distribution.
- **Scaling geospatial index**: may fit in one server; read replicas for read load.
- **Cache strategy**: cache key by geohash (list of business IDs) and business_id (details); better than raw coordinates due to GPS inaccuracy and movement.
- **Final flow**: user request → LB → LBS → geohash length from radius → neighboring geohashes → parallel Redis queries for business IDs → fetch details from business info Redis → sort by distance → return.

## Procedure
1. Define functional and non-functional requirements and scale.
2. Define APIs for search and business CRUD.
3. Choose a relational DB for business data with primary-replica.
4. Select a geospatial indexing method (geohash, quadtree, S2) based on needs.
5. Design geospatial index table and caching by geohash and business ID.
6. Implement search by converting radius to geohash precision, fetching neighbors, querying caches/DB, and ranking by distance.
7. Scale business table by ID and geospatial index with replicas if needed.
8. Deploy across regions and batch-update business data.

## Tradeoffs and failure modes
- Naive 2D search is simple but inefficient at scale.
- Geohash is easy but needs neighbor lookups to handle boundaries.
- Quadtree supports k-nearest and density adaptation but is harder to update.
- S2 is powerful for geofencing but more complex.
- Caching by geohash improves performance but needs careful key design and invalidation.

## Checks
- Is geospatial indexing chosen appropriately for the query types?
- Are boundary issues handled (e.g., neighbor searches)?
- Is caching designed with robust keys?
- Is the read-heavy path scaled with replicas and parallelism?
