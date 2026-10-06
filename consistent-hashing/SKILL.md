---
name: consistent-hashing
description: Distribute requests and data across servers with minimal reassignment on membership changes; use when designing sharding, caching, or load balancing rings.
---

# Design Consistent Hashing

## Purpose
Distribute data and requests across servers while minimizing redistribution when servers are added or removed, and mitigate hotspots through balanced distribution.

## Key concepts
- **Rehashing problem**: simple `hash(key) % N` redistributes most keys when N changes, causing cache misses and overload.
- **Hash space and ring**: hash values from 0 to 2^160-1 (e.g., SHA-1) form a continuous ring; servers are mapped onto the ring by IP or name.
- **Server lookup**: a key's server is found by moving clockwise from the key's position until a server is found.
- **Adding/removing servers**: only nearby keys are redistributed; adding affects keys between the new server and its predecessor; removing reassigns keys from the removed server to the next clockwise server.
- **Challenges**: uneven partition sizes and non-uniform key distribution.
- **Virtual nodes**: each server is represented by multiple virtual nodes spread uniformly on the ring; more virtual nodes reduce standard deviation and balance load.
- **Affected keys**: for addition, keys between predecessor and new server; for removal, keys between predecessor and removed server.

## Procedure
1. Define the hash space and ring using a hash function.
2. Map servers onto the ring by hashing server identifiers.
3. For a key, hash it and walk clockwise to find the responsible server.
4. Add or remove servers and reassign only the affected ranges.
5. Introduce virtual nodes per server to improve balance and distribution.
6. Use consistent hashing where horizontal scaling and membership changes are expected.

## Tradeoffs and failure modes
- Basic ring can produce uneven partitions and hotspots without virtual nodes.
- More virtual nodes improve balance but increase ring metadata and lookup complexity.
- Membership changes still cause some key movement; plan for cache misses during transitions.
- The hash function and ring size affect distribution quality.

## Checks
- Does membership change only affect a fraction of keys?
- Are virtual nodes used to balance load?
- Is the hash function appropriate for the key and server space?
- Are hotspots and uneven partitions addressed?
