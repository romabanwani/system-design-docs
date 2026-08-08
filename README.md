# System Design Docs

A collection of architecture write-ups, decision records, and system designs from real problems
I've worked on. This repo exists to document *how* and *why* systems are designed a certain way —
not just the code.

## Contents

| Doc | Summary |
|---|---|
| [Monolith → Microservices Migration](docs/monolith-to-microservices.md) | Migrating a legacy PHP e-commerce monolith to a Node.js-based microservices architecture, with zero-downtime cutover strategy |
| [Architecture Decision Records](docs/decisions/) | Individual ADRs documenting specific technical trade-offs made during the migration |

## Why this repo exists

Most of my production work lives in private/client repos. This is where I document the *thinking*
behind system design decisions — requirements gathering, trade-off analysis, and the reasoning
that doesn't show up in a diff. Feedback and discussion welcome via Issues.
