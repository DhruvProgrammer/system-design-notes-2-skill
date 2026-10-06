---
name: web-crawler-design
description: Design a scalable web crawler for search indexing; use when discussing crawling architecture, politeness, deduplication, frontier, and robustness.
---

# Design a Web Crawler

## Purpose
Design a scalable, robust, polite, and extensible web crawler that discovers and collects web content (e.g., for search engine indexing).

## Key concepts
- **Applications**: search indexing, web archiving, web mining, web monitoring.
- **Challenges**: scalability (billions of pages, parallelization), robustness (bad HTML, crashes, malicious links), politeness (avoid overwhelming servers), extensibility (new content types).
- **Components**:
  - **Seed URLs**: starting points, selected by locality or topic.
  - **URL frontier**: FIFO queue of URLs to download.
  - **HTML downloader**: fetches pages.
  - **DNS resolver**: resolves URLs to IPs.
  - **Content parser**: validates and parses pages; discards malformed content.
  - **Content seen?**: duplicate detection via hash comparison.
  - **Content storage**: stores HTML; popular content in memory.
  - **URL extractor**: extracts new links.
  - **URL filter**: excludes blacklisted/bad URLs.
  - **URL seen?**: tracks visited URLs.
  - **URL storage**: stores visited URLs.
- **Workflow**: seed URLs → frontier → downloader (DNS resolve) → parser → content seen → extractor → filter → frontier.
- **DFS/BFS**: web as directed graph; BFS preferred due to depth; standard BFS ignores priority.
- **Politeness**: one request per host at a time; per-host queues with delays; queue router, mapping table, queue selector, worker threads.
- **Priority**: assign higher priority via PageRank or update frequency; prioritizer, prioritized queues, biased selection.
- **Freshness**: recrawl based on update history or importance.
- **HTML downloader optimizations**: robots.txt compliance, distributed crawling, DNS cache, geographic distribution, short timeouts.
- **Robustness**: consistent hashing for load distribution, error handling, data validation.
- **Extensibility**: plug in modules for new content types.
- **Problematic content**: duplicate detection via hashes, spider trap avoidance (e.g., URL length limits), data noise filtering.

## Procedure
1. Define requirements: scale, content types, duplicate handling, storage retention.
2. Identify components: seed URLs, frontier, downloader, DNS, parser, dedup, storage, extractor, filter.
3. Implement BFS-based traversal with a URL frontier.
4. Add politeness via per-host queues and delays.
5. Add priority and freshness scheduling for important pages.
6. Ensure DNS caching, distributed crawling, and timeouts for performance.
7. Add duplicate detection and spider trap safeguards.
8. Plan for extensibility via modular content handlers.

## Tradeoffs and failure modes
- BFS is generally preferred, but priority and freshness need additional structures.
- Politeness reduces throughput to a given host but avoids overloading servers.
- Duplicate detection via hashing adds overhead but prevents redundant work.
- Distributed crawling improves scale but adds coordination complexity.
- Spider traps and bad content can cause runaway or wasted work without safeguards.

## Checks
- Is politeness enforced per host?
- Are duplicates detected and filtered?
- Is the frontier and priority model appropriate for the scale?
- Are robustness and extensibility addressed?
