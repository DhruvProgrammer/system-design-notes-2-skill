---
name: notification-system-design
description: Design a scalable notification system for push, SMS, and email; use when discussing delivery flows, queues, reliability, and third-party integration.
---

# Design a Notification System

## Purpose
Design a scalable system that sends push notifications, SMS, and emails at high volume with soft real-time delivery, reliability, and user preferences.

## Key concepts
- **Channels**: push (APNS for iOS, FCM for Android), SMS (e.g., Twilio, Nexmo), email (e.g., SendGrid, Mailchimp).
- **Contact info gathering**: collect device tokens, phone numbers, emails at signup/install; store in device token and user tables.
- **Trigger services**: microservices, cron jobs, or distributed systems that generate notification events.
- **Initial design challenges**: SPOF, scalability limits, performance bottlenecks.
- **Improved design**: move DB/cache out of notification server, add horizontal scaling, use message queues to decouple components, add workers to pull events and send via third-party services.
- **Reliability**: persist notification data and retry; deduplicate by event ID to avoid duplicates.
- **Additional components**: notification templates, notification settings (opt-in/opt-out per channel), rate limiting, retry mechanism, queue monitoring, event tracking (open/click/engagement).
- **Security**: app key/secret for API authentication.
- **Flow**: trigger services call APIs → notification servers validate and fetch metadata → events to queues → workers process and call third-party services → delivery.

## Procedure
1. Define channels, scale, delivery expectations, platforms, triggers, and opt-out needs.
2. Gather and store contact info (device tokens, phone, email).
3. Define notification APIs for trigger services.
4. Move state (DB/cache) out of notification servers and scale horizontally.
5. Introduce message queues to decouple and buffer.
6. Add workers to process events and call third-party providers.
7. Add persistence, deduplication by event ID, and retries for reliability.
8. Add templates, user settings, rate limiting, monitoring, and tracking.

## Tradeoffs and failure modes
- Queues decouple components and absorb spikes but add latency and operational complexity.
- Third-party delivery depends on external providers; retries and monitoring are important.
- Deduplication prevents spam but requires event ID tracking.
- Rate limiting protects users but must be tuned to avoid blocking legitimate notifications.

## Checks
- Are channels and scales defined?
- Is the system horizontally scalable and free of SPOF?
- Are queues and workers used to decouple delivery?
- Is data persisted and deduplicated?
- Are retries, templates, settings, and monitoring in place?
