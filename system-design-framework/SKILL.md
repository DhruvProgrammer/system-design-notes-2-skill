---
name: system-design-interview-framework
description: A 4-step framework for system design interviews; use when structuring a design discussion from requirements to wrap-up.
---

# A Framework for System Design Interviews

## Purpose
Navigate a system design interview with a structured, collaborative approach that covers requirements, high-level design, deep dives, and tradeoffs.

## Key concepts
- **Step 1 — Understand the problem and scope**: clarify requirements and assumptions, avoid premature solutions, ask good questions.
- **Step 2 — Propose high-level design and get buy-in**: draft a blueprint with key components, treat the interviewer as a teammate, do back-of-the-envelope calculations, walk through use cases.
- **Step 3 — Design deep dive**: focus on the most relevant components, discuss bottlenecks and solutions, balance depth against over-engineering.
- **Step 4 — Wrap-up**: summarize tradeoffs, identify bottlenecks and improvements, discuss scaling and error handling.

## Procedure
1. Clarify requirements: most important features, scale, platforms, existing constraints.
2. Document assumptions visibly for reference.
3. Draft a high-level blueprint with boxes for clients, APIs, databases, caches, CDNs.
4. Perform rough capacity calculations to validate the design against scale.
5. Walk through the main use cases and edge cases.
6. Deep-dive into the components most relevant to the problem.
7. Identify bottlenecks and propose mitigations.
8. Summarize the design, tradeoffs, and possible enhancements.

## Best practices
**Do**: ask questions, communicate your thinking, iterate with the interviewer, show flexibility, focus on critical components.
**Don't**: design before understanding requirements, go silent, or over-engineer unnecessarily.

## Time management (45-minute interview, approximate)
- Understand problem and scope: 3–10 minutes
- High-level design and buy-in: 10–15 minutes
- Deep dive: 10–25 minutes
- Wrap-up: 3–5 minutes

## Tradeoffs and failure modes
- Jumping into design too early risks solving the wrong problem.
- Over-deepening one area can leave the overall design weak.
- Silence reduces collaboration and visibility into your reasoning.
- Over-engineering wastes time and can obscure the core ideas.

## Checks
- Were requirements and scale clarified before designing?
- Is the high-level design validated with rough calculations?
- Are the most important components addressed in the deep dive?
- Are tradeoffs and next steps summarized at the end?
