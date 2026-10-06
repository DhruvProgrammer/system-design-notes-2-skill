---
name: youtube-design
description: Design a video streaming platform with upload, transcoding, and playback; use when discussing video upload flow, transcoding DAGs, CDNs, and cost/security optimizations.
---

# Design YouTube

## Purpose
Design a scalable video streaming platform supporting fast uploads, smooth playback, quality switching, low infrastructure cost, and high availability.

## Key concepts
- **Scale**: billions of users, billions of daily views, large ad revenue; multi-language.
- **Core features**: upload videos, watch videos; platforms: mobile, web, smart TVs.
- **Assumptions/examples**: DAU, average video size, upload limits, daily storage need, CDN costs.
- **Components**: client, CDN, API servers (non-streaming interactions), metadata database, original storage (blob), transcoding servers, transcoded storage (blob).
- **Video upload flow**: upload to original storage; transcoding servers convert to multiple formats; transcoded videos sent to transcoded storage and distributed to CDN; completion events queued; completion handlers update metadata and notify users; metadata upload in parallel.
- **Video streaming flow**: stream from CDN edge servers to minimize latency; supported streaming protocols (e.g., MPEG-DASH, HLS, HDS).
- **Video transcoding**: reduces storage, ensures compatibility, adapts quality; container (MP4, AVI) and codecs (H.264, VP9); DAG model for parallelism (split into video/audio/metadata, encoding, thumbnails, watermarking).
- **Transcoding architecture**: preprocessor (split into GOP-aligned chunks, generate DAG, store GOPs/metadata in temp storage), DAG scheduler (split into stages, put tasks in task queue), resource manager (task/worker/running queues, task scheduler), task workers, temporary storage, output.
- **Speed optimizations**: parallel/resumable chunked uploads, distributed upload centers via CDN, parallel processing with message queues.
- **Safety optimizations**: pre-signed URLs for authorized uploads, DRM/encryption/watermarking.
- **Cost optimizations**: serve popular videos via CDN, on-demand encoding for rare videos, regionalize distribution, custom CDNs/ISP partnerships.
- **Error handling**: retry recoverable errors; stop and return errors for malformed videos.

## Procedure
1. Define scope, scale, features, and assumptions.
2. Design high-level components: clients, CDN, API servers, metadata DB, original and transcoded storage, transcoding servers.
3. Design upload flow with parallel original upload and metadata update.
4. Design streaming flow from CDN.
5. Design transcoding pipeline with DAG, scheduler, resource manager, workers, and temporary storage.
6. Add speed, safety, and cost optimizations.
7. Add error handling for recoverable and non-recoverable errors.

## Tradeoffs and failure modes
- Transcoding is computationally expensive; parallelism and DAGs help but add complexity.
- CDN improves playback latency and offload but adds cost.
- Pre-signed URLs and DRM improve security but add integration complexity.
- On-demand encoding saves cost for rare videos but can increase latency for those videos.
- Temporary storage and retries add robustness but require cleanup.

## Checks
- Are upload and streaming flows efficient and parallel where possible?
- Is transcoding parallelized and resilient?
- Are security and cost optimizations appropriate for the content mix?
- Are errors handled gracefully?
