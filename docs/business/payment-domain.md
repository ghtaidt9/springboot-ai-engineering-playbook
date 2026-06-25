# Payment Domain

## Responsibilities

The Payment Domain manages:
- Payment processing
- Refund processing
- Payment status tracking

## Refund Rules

Requirements:
- Only paid orders are refundable
- Refund window is 30 days
- Partial refunds are supported
- Every refund must be audited

## Domain Events

Produces:
- PaymentCompleted
- RefundProcessed

Consumes:
- OrderCreated