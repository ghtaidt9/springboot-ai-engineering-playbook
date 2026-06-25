# Backend Architecture

## Architectural Style

The backend follows a Layered Architecture.

Request Flow:

Controller
↓
Service
↓
Repository
↓
PostgreSQL

## Layer Responsibilities

### Controller Layer

Responsibilities:

- REST API endpoints
- Request validation
- DTO conversion
- HTTP status handling

Must NOT:

- Contain business logic
- Access repositories directly

---

### Service Layer

Responsibilities:

- Business logic
- Transaction boundaries
- Domain orchestration

Must NOT:

- Return entities directly
- Contain HTTP concerns

---

### Repository Layer

Responsibilities:

- Data access
- Persistence operations

Implementation:

Mybatis

Must NOT:

- Contain business rules

---

## API Standards

Style:

RESTful API

Conventions:

GET    /api/orders/{id}
POST   /api/orders
PUT    /api/orders/{id}
DELETE /api/orders/{id}

Response:

Use standardized API response objects.

Errors:

Use GlobalExceptionHandler.