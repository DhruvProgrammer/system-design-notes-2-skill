---
name: google-maps-design
description: Design a mapping service with location updates, navigation, ETA, and map rendering; use when discussing map tiles, routing tiles, geocoding, hierarchical routing, and traffic.
---

# Design Google Maps

## Purpose
Design a mapping service supporting location updates, navigation with ETA, and map rendering at large scale, with accuracy, smooth rendering, and efficient data/battery use.

## Key concepts
- **Scope**: DAU, features (location update, navigation, ETA, map rendering), road data size, traffic consideration, travel modes, multi-stop scope, business places.
- **Non-functional**: accuracy, smooth navigation, low data/battery usage, availability, scalability.
- **Map concepts**:
  - **Positioning**: latitude/longitude on a sphere.
  - **Map projection**: 3D to 2D; distortions; Web Mercator used by Google Maps.
  - **Geocoding**: address ↔ coordinates; interpolation from street network data.
  - **Geohashing**: encodes geographic area into string; recursive quadrants.
  - **Map rendering via tiling**: world broken into tiles; client downloads relevant tiles by location and zoom; different tiles per zoom level.
  - **Road data for routing**: intersections as nodes, roads as edges; modified Dijkstra/A*; graph too large for whole world; use routing tiles with hierarchical detail; tiles reference neighbors; reduces memory bandwidth.
- **Estimation**: world map storage (~70PB with compression), metadata negligible, road info as routing tiles; navigation QPS from DAU and usage; batched GPS updates; peak QPS.
- **High-level design**:
  - **Location service**: record location updates every t seconds; batched updates; stream for traffic/analysis; Cassandra for heavy writes; Kafka for streaming.
  - **Navigation service**: find fast routes; not necessarily fastest but accurate; supports geocoding, route planning, shortest-path (A* on routing tiles), ETA via ML, ranker for filters, updater to keep DBs fresh.
  - **Map rendering**: tiles fetched on-demand by location/zoom; static tiles served from CDN by geohash; client-side tile URL calculation vs API; vector tiles for bandwidth savings.
- **Data model**:
  - **Routing tiles**: offline pipeline from raw road data; store in S3, cache aggressively; compress adjacency lists.
  - **User location data**: Cassandra with user_id partition key and timestamp clustering key; write-heavy, availability over consistency.
  - **Geocoding DB**: Redis for fast read, infrequent writes.
  - **Precomputed map tiles**: CDN.
- **Location service deep dive**: batch updates, Cassandra row design, Kafka streaming for analysis.
- **Rendering optimizations**: vector tiles reduce bandwidth.
- **Navigation deep dive**: geocoding, route planner, shortest-path on routing tiles, ETA service, ranker, updater; adaptive ETA and rerouting by storing navigating users' tiles and checking traffic events; store origin tile and several super tiles to reduce DB size; keep multiple possible routes for faster reroute.
- **Delivery protocols**: mobile push limited; WebSocket better than long-polling for compute; SSE possible; WebSocket for bi-directional needs.
- **Wrap-up**: final design; multi-stop navigation as enterprise feature.

## Procedure
1. Define scope, features, scale, and non-functional requirements.
2. Understand map concepts: projection, geocoding, geohashing, tiling, routing tiles, hierarchical routing.
3. Estimate storage and QPS.
4. Design location service with batching, Cassandra, and Kafka streaming.
5. Design navigation service with geocoding, shortest-path on routing tiles, ETA, ranking, and updater.
6. Design map rendering with static CDN tiles and optional vector tiles.
7. Add adaptive ETA/rerouting and delivery protocol choices.
8. Add data model and storage choices for routing tiles, location, geocoding, and tiles.

## Tradeoffs and failure modes
- Tiling and hierarchical routing reduce memory/bandwidth but add complexity.
- Batched location updates reduce load but add latency for real-time uses.
- Static CDN tiles are simple and cacheable; vector tiles save bandwidth but require client rendering.
- Adaptive rerouting improves UX but requires tracking navigating users and traffic events.
- Delivery protocol choice balances latency, bi-directionality, and client support.

## Checks
- Are location updates batched and stored appropriately?
- Is routing using hierarchical tiles efficient for the scale?
- Are map tiles served with low latency via CDN?
- Is adaptive rerouting and traffic integration addressed?
