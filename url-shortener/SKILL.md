---
name: url-shortener-design
description: Design a URL shortening service with redirect and analytics; use when discussing short URL generation, redirect types, storage, and scaling.
---

# Design a URL Shortener

## Purpose
Build a scalable URL shortening service that generates unique, short URLs, redirects users, and supports high read traffic and analytics.

## Key concepts
- **Requirements**: unique and short URLs, handle ~100M new URLs/day over 10 years, ~10:1 read-to-write, large storage needs.
- **API endpoints**: POST for shortening (longUrl → shortURL), GET for redirect (shortURL → longURL).
- **Redirect types**: 301 permanent (browser-cached, fewer future requests to service) vs 302 temporary (useful for analytics).
- **Data model**: store <shortURL, longURL> mappings in a relational table (id, shortURL, longURL).
- **Short URL generation**:
  - **Base62 conversion**: encode a unique ID using [0-9a-zA-Z]; 7 chars supports ~3.5 trillion URLs; needs a unique ID generator; length grows with ID; predictable next URL possible.
  - **Hash + collision resolution**: use CRC32/MD5/SHA-1; take first chars; collisions possible; resolve recursively or with Bloom filters; fixed length; no ID generator needed.
- **Shortening flow**: check if longURL exists; if yes return existing shortURL; else generate unique ID, convert to shortURL via Base62, store mapping.
- **Redirect flow**: on click, check cache first, then database; redirect to longURL.
- **Additional concerns**: rate limiting, analytics, replication, sharding, high availability.

## Procedure
1. Define requirements: scale, read/write ratio, redirect behavior, analytics.
2. Define API endpoints for shorten and redirect.
3. Choose redirect type based on caching and analytics needs.
4. Design the data model for mappings.
5. Choose a short URL generation strategy: Base62 of unique ID vs hash with collision resolution.
6. Implement shortening flow with deduplication of long URLs.
7. Implement redirect flow with caching for performance.
8. Add rate limiting, replication, sharding, and analytics as needed.

## Tradeoffs and failure modes
- 301 reduces future requests but limits analytics; 302 enables analytics but increases service load.
- Base62 needs an ID generator but avoids collisions and is predictable; hash-based needs collision handling and may be expensive to resolve.
- Caching improves redirect latency but adds consistency considerations.
- High read-to-write ratio favors cache-heavy designs and read replicas.

## Checks
- Are short URLs unique and appropriately short?
- Is redirect behavior (301 vs 302) chosen deliberately?
- Is the generation strategy matched to scale and collision needs?
- Is caching used for redirects?
- Are rate limiting and analytics considered?
