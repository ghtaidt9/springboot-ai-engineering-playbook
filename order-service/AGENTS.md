# AGENTS.md — order-service

> Nested hub for `order-service`. Root `AGENTS.md` applies in full.
> This file adds scope-specific routing for this service only.

---

## Service Responsibility

`order-service` **owns**: Order aggregate, OrderItem, order lifecycle events.

`order-service` **does NOT own**: Payment processing, inventory reservation, notification delivery.

---

## Context Routing (order-service specific)

| Task | Read |
|------|------|
| Order business rules | `docs/business/order-domain.md` |
| Domain events this service publishes | `docs/business/order-domain.md` → "Domain Events" section |
| Events this service **consumes** | `PaymentConfirmedEvent` from `payment-service` |
| DB schema for this service | `src/main/resources/db/migration/` |

---

## Package Root

```
com.example.order
```

## Key Domain Events

**Publishes:**
- `OrderCreatedEvent` → triggers payment initiation
- `OrderConfirmedEvent` → triggers inventory deduction, notification
- `OrderCancelledEvent` → triggers inventory release
- `OrderShippedEvent` → triggers notification
- `OrderDeliveredEvent` → triggers notification

**Consumes:**
- `PaymentConfirmedEvent` (from payment-service) → call `order.confirm()`
- `PaymentFailedEvent` (from payment-service) → call `order.cancel(reason)`

---

## Hard Constraints (service-specific)

- NEVER read `payments` table — consume events only
- NEVER read `inventory` table — consume events only
- Order total is always recomputed from items — never trust request total
