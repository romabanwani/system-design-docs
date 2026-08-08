# ADR-0001: Use Strangler Fig Pattern for Monolith Migration

**Status:** Accepted
**Date:** 2026-01-15

## Context

We need to migrate a PHP monolith to a microservices architecture without stopping feature
development or risking prolonged downtime. The team is mid-sized (6 engineers) and cannot
afford a long feature freeze.

## Decision

We will use the **Strangler Fig pattern**: introduce an API gateway in front of the monolith,
and incrementally extract and route individual domains (catalog, orders, notifications) to new
Node.js services one at a time, deleting the corresponding monolith code once each extraction
is stable in production.

Inventory will remain on PHP/MySQL for now — it's tightly coupled to a third-party ERP
integration that isn't worth re-integrating in phase one.

## Alternatives Considered

1. **Full rewrite ("big bang")** — rejected. Too risky for a live revenue-generating system;
   no incremental value delivery; rollback would mean reverting weeks of work at once.
2. **Leave the monolith as-is, only optimize** — rejected. Doesn't address the core problem:
   teams cannot deploy or scale independently.

## Consequences

**Positive**
- Every extraction is independently shippable and reversible
- Team can prioritize the highest-pain domain first (order processing)
- No prolonged feature freeze

**Negative**
- Temporary added complexity: routing logic in the gateway, and running two systems in parallel
  during the transition
- Requires discipline to actually delete monolith code after each extraction, or the "temporary"
  complexity becomes permanent
