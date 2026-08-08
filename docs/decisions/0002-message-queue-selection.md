# ADR-0002: RabbitMQ for Asynchronous Event Handling

**Status:** Accepted
**Date:** 2026-01-22

## Context

Order placement needs to trigger downstream side effects (email confirmation, SMS, inventory
sync) without making the checkout request wait on all of them. We need a message broker to
decouple the Order Service from these consumers.

## Decision

We will use **RabbitMQ** with a topic exchange, where the Order Service publishes an
`OrderPlaced` event and the Notification Service (and others) subscribe independently.

## Alternatives Considered

| Option | Notes |
|---|---|
| **RabbitMQ** (chosen) | Mature, easy to self-host or use managed (CloudAMQP), good client support in both Node.js and PHP for the transition period, sufficient throughput for our scale (~thousands of orders/day, not millions) |
| Kafka | Better for very high-throughput event streaming and replay, but operationally heavier than we need at current scale; revisit if order volume grows 50-100x |
| Redis Pub/Sub | Simple, but no durable delivery guarantee — a consumer offline during a publish would silently miss the event, unacceptable for order confirmations |
| Direct HTTP calls between services | Rejected — creates tight coupling and cascading failure risk (if Notification Service is down, checkout would fail) |

## Consequences

**Positive**
- Order Service stays fast and doesn't block on notification/downstream logic
- New consumers (e.g. future analytics service) can subscribe to `OrderPlaced` without any
  change to the Order Service
- Both legacy PHP services and new Node.js services have solid RabbitMQ client libraries,
  useful during the transition period

**Negative**
- Introduces an additional piece of infrastructure to operate and monitor
- Requires idempotent consumers, since queues can occasionally deliver a message more than once
- Team needs to build monitoring/alerting for queue depth and dead-letter handling
