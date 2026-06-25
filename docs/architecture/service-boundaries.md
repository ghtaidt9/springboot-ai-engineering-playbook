# Service Boundaries

## Order Service
Owns:
- Orders
- Order status
- Order history
Must not:
- Process payments
- Access payment database

---

## Payment Service
Owns:
- Payments
- Refunds
Must not:
- Modify orders directly

---

## User Service
Owns:
- User profiles
- Authentication
- User preferences