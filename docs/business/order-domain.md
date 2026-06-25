# Order Domain

## Scope
`order-service` only. Defines business rules, state machine, and events for the Order aggregate.

---

## Rules

MUST:
- An order MUST have at least one `OrderItem` at creation
- `customerId` and each `productId` MUST be non-blank strings
- Total amount is always computed from items — never stored independently
- Status transitions MUST follow the state machine below
- `OrderCreatedEvent` MUST be published after successful `save()`

MUST NOT:
- Create an order with a negative or zero quantity
- Skip the `PENDING` → `CONFIRMED` step (no direct `PENDING` → `SHIPPED`)
- Modify order items after status is `CONFIRMED`

---

## State Machine

```
PENDING ──[confirm payment]──► CONFIRMED ──[dispatch]──► SHIPPED ──[deliver]──► DELIVERED
   │                               │                         │
   └──[cancel]──► CANCELLED        └──[cancel]──► CANCELLED  │
                                                             └──[return]──► RETURNED
```

Valid transitions:
| From | Event | To |
|------|-------|----|
| `PENDING` | `confirm` | `CONFIRMED` |
| `PENDING` | `cancel` | `CANCELLED` |
| `CONFIRMED` | `dispatch` | `SHIPPED` |
| `CONFIRMED` | `cancel` | `CANCELLED` |
| `SHIPPED` | `deliver` | `DELIVERED` |
| `SHIPPED` | `return` | `RETURNED` |

---

## Domain Events Published

| Event | Trigger | Key Payload Fields |
|-------|---------|-------------------|
| `OrderCreatedEvent` | Order saved in PENDING | `orderId`, `customerId`, `totalAmount`, `itemIds` |
| `OrderConfirmedEvent` | Status → CONFIRMED | `orderId`, `customerId` |
| `OrderCancelledEvent` | Status → CANCELLED | `orderId`, `reason` |
| `OrderShippedEvent` | Status → SHIPPED | `orderId`, `trackingNumber` |
| `OrderDeliveredEvent` | Status → DELIVERED | `orderId`, `deliveredAt` |

---

## Examples

```java
// Creating an order (domain method)
public static Order create(CustomerId customerId, List<OrderItem> items) {
    if (items == null || items.isEmpty()) {
        throw new DomainException("Order must have at least one item");
    }
    return new Order(OrderId.generate(), customerId, items, OrderStatus.PENDING, Instant.now());
}

// State transition (domain method — guards enforced here)
public void confirm() {
    if (this.status != OrderStatus.PENDING) {
        throw new DomainException("Cannot confirm order in status: " + this.status);
    }
    this.status = OrderStatus.CONFIRMED;
}
```

---

## Related
- `docs/business/payment-domain.md` — payment triggers `OrderConfirmedEvent` consumer
- `docs/architecture/event-driven-architecture.md` — event publishing pattern
- `docs/adr/ADR-001.md` — SQS FIFO decision
- `.cursor/rules/events-sqs.mdc` — event code patterns

## Out of Scope
- Payment processing logic (see `payment-domain.md`)
- Inventory deduction (see `inventory-service`)
- Notification delivery (see `notification-service`)
