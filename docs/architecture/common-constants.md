# Common Constants & Messages

## Scope
All services. Defines where constants, error codes, and messages live,
how they are named, and what belongs in shared code vs service-specific code.

---

## Package Structure

```
src/main/java/com/example/{service}/
  common/
    constant/
      ErrorCode.java          # String constants for ApiErrorResponse.error field
      EventType.java          # String constants for DomainEvent.eventType field
    exception/
      DomainException.java    # 422 — business rule violated
      NotFoundException.java  # 404 — resource not found
      ConflictException.java  # 409 — duplicate or state conflict
      GlobalExceptionHandler.java
    dto/
      response/
        ApiErrorResponse.java # record — error envelope
        PageResponse.java     # record — paginated list wrapper
```

No shared library between services — each service has its own copy of `common/`.
If a service needs a constant from another service, it redefines it locally.

---

## ErrorCode.java

```java
// common/constant/ErrorCode.java
public final class ErrorCode {
    private ErrorCode() {}

    // 400
    public static final String VALIDATION_FAILED      = "VALIDATION_FAILED";

    // 404
    public static final String RESOURCE_NOT_FOUND     = "RESOURCE_NOT_FOUND";

    // 409
    public static final String CONFLICT               = "CONFLICT";

    // 422
    public static final String BUSINESS_RULE_VIOLATION = "BUSINESS_RULE_VIOLATION";

    // 500
    public static final String INTERNAL_ERROR         = "INTERNAL_ERROR";
}
```

---

## EventType.java (per service — only events this service publishes)

```java
// common/constant/EventType.java  (in order-service)
public final class EventType {
    private EventType() {}

    public static final String ORDER_CREATED   = "OrderCreated";
    public static final String ORDER_CONFIRMED = "OrderConfirmed";
    public static final String ORDER_CANCELLED = "OrderCancelled";
    public static final String ORDER_SHIPPED   = "OrderShipped";
    public static final String ORDER_DELIVERED = "OrderDelivered";
}
```

---

## Exception Classes

```java
// DomainException → 422
public class DomainException extends RuntimeException {
    public DomainException(String message) {
        super(message);
    }
}

// NotFoundException → 404
public class NotFoundException extends RuntimeException {
    public NotFoundException(String message) {
        super(message);
    }
}

// ConflictException → 409
public class ConflictException extends RuntimeException {
    public ConflictException(String message) {
        super(message);
    }
}
```

---

## GlobalExceptionHandler.java

```java
@RestControllerAdvice
@RequiredArgsConstructor
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiErrorResponse> handleValidation(
            MethodArgumentNotValidException ex, HttpServletRequest req) {
        String message = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + " " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(error(400, ErrorCode.VALIDATION_FAILED, message, req));
    }

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ApiErrorResponse> handleNotFound(
            NotFoundException ex, HttpServletRequest req) {
        return ResponseEntity.status(404).body(error(404, ErrorCode.RESOURCE_NOT_FOUND, ex.getMessage(), req));
    }

    @ExceptionHandler(DomainException.class)
    public ResponseEntity<ApiErrorResponse> handleDomain(
            DomainException ex, HttpServletRequest req) {
        return ResponseEntity.status(422).body(error(422, ErrorCode.BUSINESS_RULE_VIOLATION, ex.getMessage(), req));
    }

    @ExceptionHandler(ConflictException.class)
    public ResponseEntity<ApiErrorResponse> handleConflict(
            ConflictException ex, HttpServletRequest req) {
        return ResponseEntity.status(409).body(error(409, ErrorCode.CONFLICT, ex.getMessage(), req));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiErrorResponse> handleGeneric(
            Exception ex, HttpServletRequest req) {
        log.error("Unhandled exception", ex);
        return ResponseEntity.status(500).body(error(500, ErrorCode.INTERNAL_ERROR, "Unexpected error", req));
    }

    private ApiErrorResponse error(int status, String code, String message, HttpServletRequest req) {
        return new ApiErrorResponse(
            Instant.now(), status, code, message,
            req.getRequestURI(), MDC.get("traceId")
        );
    }
}
```

---

## Message Strings — Where They Live

**Inline trong exception throw** — không dùng message properties file trừ khi cần i18n:

```java
// Good — inline, dễ đọc, dễ tìm
throw new NotFoundException("Order not found: " + orderId);
throw new DomainException("Cannot confirm order in status: " + order.getStatus());
throw new ConflictException("Order already confirmed: " + orderId);

// Avoid — over-engineering nếu chưa cần i18n
throw new DomainException(MessageConstants.ORDER_ALREADY_CONFIRMED);
```

Nếu project cần i18n sau này → tạo `messages.properties` và ADR riêng.

---

## Rules

MUST:
- `ErrorCode` constants là `public static final String` trong class `final` với private constructor
- `EventType` constants chỉ chứa events mà service đó **publish** — không copy events của service khác
- Mọi exception đều có `GlobalExceptionHandler` mapping — không để 500 leak ra ngoài
- `common/` package tự chứa trong mỗi service — không shared jar

MUST NOT:
- Dùng magic string cho `error` field trong response — luôn dùng `ErrorCode.*`
- Dùng magic string cho `eventType` — luôn dùng `EventType.*`
- Tạo `MessageConstants` class cho message strings (trừ khi có i18n requirement)
- Throw `RuntimeException` trực tiếp — luôn dùng typed exception

---

## Related
- `docs/architecture/api-response-contract.md` — response shapes
- `.cursor/rules/api-rest.mdc` — controller + error format
- `.cursor/rules/events-sqs.mdc` — event envelope structure
