# API Response Contract

## Scope
All services. Defines exact JSON shape for success, error, and paginated responses.
Agent MUST follow these shapes — do not invent custom wrappers per feature.

---

## Success Response

### Single resource (GET one, POST, PUT, PATCH)
No wrapper — return the DTO directly.

```json
// POST /api/v1/orders → 201
{
  "orderId": "550e8400-e29b-41d4-a716-446655440000",
  "customerId": "customer-123",
  "status": "PENDING",
  "totalAmount": 150.00,
  "currency": "USD",
  "items": [
    {
      "orderItemId": "...",
      "productId": "product-456",
      "quantity": 2,
      "unitPrice": 75.00,
      "subtotal": 150.00
    }
  ],
  "createdAt": "2026-06-25T10:00:00Z"
}
```

### Collection / paginated (GET list)
Wrap in `PageResponse<T>`:

```json
// GET /api/v1/orders?page=0&size=20 → 200
{
  "content": [ { ...order... }, { ...order... } ],
  "page": 0,
  "size": 20,
  "totalElements": 45,
  "totalPages": 3,
  "first": true,
  "last": false
}
```

### Empty success (DELETE)
No body — 204 No Content.

---

## Error Response

All errors use `ApiErrorResponse`:

```json
// 400 Validation Failed
{
  "timestamp": "2026-06-25T10:00:00Z",
  "status": 400,
  "error": "VALIDATION_FAILED",
  "message": "customerId must not be blank",
  "path": "/api/v1/orders",
  "traceId": "abc-123"
}

// 404 Not Found
{
  "timestamp": "2026-06-25T10:00:00Z",
  "status": 404,
  "error": "RESOURCE_NOT_FOUND",
  "message": "Order not found: 550e8400",
  "path": "/api/v1/orders/550e8400",
  "traceId": "abc-124"
}

// 422 Business Rule Violation
{
  "timestamp": "2026-06-25T10:00:00Z",
  "status": 422,
  "error": "BUSINESS_RULE_VIOLATION",
  "message": "Cannot confirm order in status: CANCELLED",
  "path": "/api/v1/orders/550e8400/confirm",
  "traceId": "abc-125"
}
```

### HTTP Status → Error Code mapping (use exactly these)

| Scenario | HTTP | `error` field |
|----------|------|---------------|
| Bean Validation fail | 400 | `VALIDATION_FAILED` |
| Missing required param | 400 | `VALIDATION_FAILED` |
| Resource not found | 404 | `RESOURCE_NOT_FOUND` |
| Business rule violation | 422 | `BUSINESS_RULE_VIOLATION` |
| Duplicate / conflict | 409 | `CONFLICT` |
| Unexpected server error | 500 | `INTERNAL_ERROR` |

---

## Java Types

```java
// common/dto/response/PageResponse.java
public record PageResponse<T>(
    List<T> content,
    int page,
    int size,
    long totalElements,
    int totalPages,
    boolean first,
    boolean last
) {
    public static <T> PageResponse<T> from(Page<T> springPage) {
        return new PageResponse<>(
            springPage.getContent(),
            springPage.getNumber(),
            springPage.getSize(),
            springPage.getTotalElements(),
            springPage.getTotalPages(),
            springPage.isFirst(),
            springPage.isLast()
        );
    }
}

// common/dto/response/ApiErrorResponse.java
public record ApiErrorResponse(
    Instant timestamp,
    int status,
    String error,     // use ErrorCode constants
    String message,
    String path,
    String traceId
) {}
```

---

## Rules

MUST:
- Single resource → return DTO directly, no wrapper
- List endpoint → wrap in `PageResponse<T>`
- All errors → use `ApiErrorResponse` with exact `error` codes from table above
- `message` field is human-readable, `error` field is machine-readable constant
- `traceId` pulled from `MDC.get("traceId")`

MUST NOT:
- Invent custom wrapper like `ApiResponse<T>` with `success: true/data/code` pattern
- Return `null` fields — omit with `@JsonInclude(NON_NULL)` if field is optional
- Use different error shapes per service

---

## Related
- `.cursor/rules/api-rest.mdc` — controller patterns
- `common/exception/GlobalExceptionHandler.java` — maps exceptions to ApiErrorResponse
- `common/constant/ErrorCode.java` — string constants for `error` field
