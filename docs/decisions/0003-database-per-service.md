# ADR-0003: Database-per-Service Instead of Shared Database

**Status:** Accepted
**Date:** 2026-01-29

## Context

The monolith uses a single shared MySQL database across all domains. This is one of the root
causes of tight coupling: a schema change for orders can silently break catalog code, and teams
can't evolve their data models independently.

## Decision

Each new microservice gets its **own database**, chosen per its access patterns:
- Catalog Service → PostgreSQL (good full-text search support for product search)
- Order Service → PostgreSQL (strong consistency needed for orders/payments)
- Inventory (legacy, retained) → MySQL, unchanged for now

Cross-service data needs are served via API calls (for reads) or event consumption (for
eventually-consistent projections), never direct database access across service boundaries.

## Alternatives Considered

1. **Keep shared database, split services on top of it** — rejected. This is a common
   anti-pattern ("distributed monolith") — you get microservice deployment complexity without
   the actual decoupling benefit, since a schema change still risks breaking other services.
2. **Shared database with strict schema ownership conventions** — rejected as an interim step
   for this project; relies on discipline rather than enforced boundaries, easy to erode over time.

## Consequences

**Positive**
- Each team can evolve their schema independently
- Failure isolation — a database issue in one service doesn't cascade to others
- Enables choosing the right database technology per service's needs

**Negative**
- No more cheap cross-domain JOINs — some queries that were a single SQL JOIN in the monolith
  now require an API call or a denormalized read model
- Eventual consistency between services must be explicitly designed for (e.g. inventory counts
  shown on the catalog page may lag by seconds after a purchase)
- More infrastructure to provision and back up
