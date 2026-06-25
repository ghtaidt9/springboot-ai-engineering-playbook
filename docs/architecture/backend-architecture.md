# Backend Architecture

## Scope
Applies to all four services: `order-service`, `payment-service`, `notification-service`, `inventory-service`.

---

## Architecture Model: Hexagonal with Internal Layering

**One sentence:** The outer boundary is Hexagonal (ports & adapters); inside each adapter the code follows standard layering.

```
┌─────────────────────────────────────────────────────────┐
│                    ADAPTERS (in)                        │
│  REST Controller  │  SQS Listener  │  Scheduled Job     │
└───────────────────────────┬─────────────────────────────┘
                            │ calls Port (interface)
┌───────────────────────────▼─────────────────────────────┐
│              APPLICATION LAYER                          │
│  OrderApplicationService  (orchestrates use cases)      │
└───────────────────────────┬─────────────────────────────┘
                            │ calls Domain
┌───────────────────────────▼─────────────────────────────┐
│                  DOMAIN LAYER                           │
│  Order  │  OrderItem  │  Money  │  OrderRepository(port)│
│  (zero framework dependencies)                          │
└───────────────────────────┬─────────────────────────────┘
                            │ implemented by
┌───────────────────────────▼─────────────────────────────┐
│                ADAPTERS (out)                           │
│  OrderRepositoryAdapter (JPA)  │  SqsEventPublisher     │
└─────────────────────────────────────────────────────────┘
```

---

## Package Structure (per service)

```
order-service/
  src/main/java/com/example/order/
    domain/
      model/          # Order, OrderItem, OrderId, Money (pure Java)
      port/
        in/           # use case interfaces: CreateOrderUseCase
        out/          # driven port interfaces: OrderRepository, OrderEventPublisher
    application/
      service/        # OrderApplicationService (implements in-ports)
      mapper/         # MapStruct DTO ↔ domain mappers
    adapter/
      in/
        web/          # OrderController, request/response records
        messaging/    # SqsOrderListener
      out/
        persistence/  # OrderRepositoryAdapter, OrderJpaEntity, OrderJpaRepository
        messaging/    # SqsOrderEventPublisher
    config/           # Spring @Configuration classes
```

---

## Rules

MUST:
- Domain classes import ONLY Java standard library and other domain classes
- Application services implement inbound port interfaces
- Repository adapters implement outbound port interfaces
- Spring annotations stay in `adapter/` and `config/` packages

MUST NOT:
- Put `@Entity` or `@Repository` in `domain/` package
- Put business logic in `@RestController`
- Have `adapter/` classes call each other directly (go through domain)

---

## Examples

**Port (domain layer):**
```java
// domain/port/out/OrderRepository.java
public interface OrderRepository {
    Order save(Order order);
    Optional<Order> findById(OrderId id);
    List<Order> findByCustomerId(CustomerId customerId);
}
```

**Adapter (out — persistence):**
```java
// adapter/out/persistence/OrderRepositoryAdapter.java
@Component
public class OrderRepositoryAdapter implements OrderRepository {
    // JPA implementation here
}
```

---

## Related
- ADR-003: Layered inside Hexagonal decision
- `.cursor/rules/java-spring.mdc`: Java conventions
- `.cursor/rules/persistence-jpa.mdc`: JPA patterns
- `docs/database/persistence.md`: database detail

## Out of Scope
- Inter-service communication protocol (see `service-boundaries.md`)
- AWS deployment topology (see `docs/aws/`)
