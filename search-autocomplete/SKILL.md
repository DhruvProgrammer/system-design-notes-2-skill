---
name: search-autocomplete-design
description: Design a search autocomplete/typeahead system; use when discussing trie-based suggestion, frequency aggregation, caching, sharding, and real-time updates.
---

# Design a Search Autocomplete System

## Purpose
Design a typeahead service that returns up to ~5 popular suggestions quickly based on query prefix and popularity, with low latency and scalability.

## Key concepts
- **Features**: up to 5 suggestions, based on query popularity, e.g., lowercase English, <100ms response, scalable.
- **Requirements**: real-time suggestions, top-k by popularity, scalability (e.g., 10M DAU, peak QPS ~48k), high availability, data growth.
- **High-level services**:
  - **Data gathering service**: collect and aggregate queries for frequency analysis; real-time processing is a starting point but often impractical at scale.
  - **Query service**: return top-k suggestions for a prefix.
- **Data gathering**: aggregate query data from analytics logs; update frequency table; build trie periodically (e.g., weekly).
- **Query service**: use frequency table and trie; lookup prefix, retrieve top-k; cache and optimize lookups; limit prefix length.
- **Trie**: tree for prefix storage; compact; stores popularity at nodes; get top-k by finding prefix node, traversing subtree, sorting children; cache top-k at each node; limit prefix length (e.g., 50).
- **Trie operations**: create weekly from aggregated data; rarely updated in real-time; delete via filters (e.g., hate speech) with async physical removal.
- **Query processing**: prefix search, top-k sorting (cached), response construction.
- **Optimizations**: cache top-k per node, limit prefix length, AJAX, browser caching.
- **Data gathering pipeline**: analytics logs (append-only, not indexed) → aggregators → frequency tables → workers rebuild trie → storage (trie cache in memory, trie DB as document store or key-value store mapping prefixes to node data).
- **Scalability**: shard trie by prefix ranges (e.g., a-m, n-z) and further within prefixes; shard map manager for routing.
- **Advanced features**: multi-language via Unicode and country-specific tries; trending queries via dynamic updates or recency weighting.

## Procedure
1. Define requirements: suggestion count, popularity basis, supported characters, latency, scale, availability.
2. Split into data gathering and query services.
3. Aggregate queries into frequency tables; rebuild trie periodically.
4. Use a trie with popularity and cached top-k at nodes.
5. Implement prefix lookup, subtree traversal, top-k selection.
6. Optimize with caching, prefix length limits, and browser caching.
7. Store trie in cache and persistent DB; map prefixes to node data.
8. Shard trie by prefix ranges and use a shard map manager.
9. Add filtering, multi-language, and trending query support as needed.

## Tradeoffs and failure modes
- Real-time trie updates on every query are impractical at scale; periodic rebuild is common.
- Caching top-k speeds lookups but requires invalidation on rebuild.
- Sharding improves scale but adds routing complexity and can complicate global top-k.
- Filtering removes unwanted suggestions but needs async cleanup and rule management.

## Checks
- Is suggestion retrieval fast enough (<100ms)?
- Is the trie built and updated at an appropriate cadence?
- Are shards and routing designed for the expected query distribution?
- Are filtering and advanced features addressed?
