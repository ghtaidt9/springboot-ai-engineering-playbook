# Service Boundaries

## Scope
All four services. Defines ownership, inter-service communication, and what each service must NOT do.

---

## Services Overview

| Service | Owns | DB Tables | Publishes | Consumes |
|---------|------|-----------|-----------|----------|
| `order-service` | Order aggregate | `orders`, `order_items` | `OrderCreated*`, `OrderConfirmed*`, `OrderCancelled*`, `OrderShipped*`, `OrderDelivered*` | `PaymentConfirmed`, `PaymentFailed` |
| `payment-service` | Payment transactions | `payments`, `payment_methods` | `PaymentConfirmed`, `PaymentFailed`, `RefundIssued` | `OrderCreated` |
| `inventory-service` | Stock levels | `inventory`, `reservations` | `InventoryReserved`, `InventoryReleased`, `StockLow` | `OrderConfirmed`, `OrderCancelled` |
| `notification-service` | Notification delivery | `notifications`, `templates` | `NotificationSent`, `NotificationFailed` | `OrderConfirmed`, `OrderShipped`, `OrderDelivered`, `PaymentFailed` |

---

## Rules

MUST:
- Each service has its own dedicated Postgres database (database-per-service)
- Cross-service data retrieval uses REST API calls or async event consumption
- Service boundary violations MUST go through an ADR before implementation

MUST NOT:
- `order-service` read `payments.*` table
- `payment-service` read `orders.*` table
- Any service write to another service's database
- Synchronous REST calls in the critical path where async events suffice

---

## Inter-Service Communication Patterns

| Pattern | When to use |
|---------|------------|
| SQS FIFO events | State changes, notifications, eventual consistency acceptable |
| REST (sync) | Query operations where caller needs immediate response |
| REST (sync) | `inventory-service` availability check at order creation |

---

## Examples

**Correct (event-driven):**
```
order-service publishes OrderCreatedEvent
    → payment-service consumes → initiates payment
    → payment-service publishes PaymentConfirmedEvent
    → order-service consumes → sets order CONFIRMED
```

**Incorrect (direct DB access):**
```
order-service SELECT * FROM payments WHERE order_id = ?  ❌
```

---

## Related
- `docs/adr/ADR-001.md` — SQS FIFO choice
- `docs/architecture/event-driven-architecture.md` — event schema
- `order-service/AGENTS.md`, `payment-service/AGENTS.md` — per-service detail

## Out of Scope
- API Gateway routing rules (see `docs/aws/api-gateway.md`)
- Authentication between services (see `docs/architecture/security.md`)
