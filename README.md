# System Design Interview Prep

This repository contains system design interview preparation materials and reference designs.

## Included content

For a single entry point to the architecture diagrams, see the
[visualization map](docs/visualizations.md).

- `docs/google-news-system-design.md` — a detailed system design reference for building a Google News–style service.
- `docs/centralized-logging-system-design.md` — a high-level design guide for a centralized logging and observability platform.
- `docs/fraudulent-card-detection-design.md` — a fraud detection design for book-now-pay-later credit card checks.
- `docs/hotel-booking-system-design.md` — a hotel booking system design with schema, scalability, and concurrency.

- `docs/dynamic-config-system-design.md` — a design summary for a read-heavy configuration service using cache-first reads, CDC-based invalidation, and object storage for large payloads.

## Purpose

Use this repo to study architectural patterns, requirements gathering, component design, scalability considerations, and trade-offs for interview questions.

## How to use

1. Read the system design reference in `docs/google-news-system-design.md`.
2. Review the key components, data flow, and scaling strategies.
3. Practice explaining the design aloud and identifying optimizations.

## Suggested follow-up topics

- Search and ranking
- News ingestion pipelines
- Caching and consistency
- API design and pagination
- Distributed systems and fault tolerance
