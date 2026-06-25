# Order Domain

## Responsibilities

The Order Domain manages:

- Order creation
- Order lifecycle
- Order cancelation
- Order state transitions

## Order states

- CREATED
- PAID
- SHIPED
- DELIVERED
- CANCELLED

## Cancellation Policy

Orders may be cancelled only when:
- Created within 24 hours
- Not shipped
- Not fully refunded

## Domain Events

Produces:
- OrderCreated
- OrderCancelled

Comsumes:
- PaymentCompleted
- RefundProcessed
