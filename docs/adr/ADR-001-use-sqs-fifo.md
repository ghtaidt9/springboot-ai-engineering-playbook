# ADR-001

## Title
Use AWS SQS FIFO for Domain Events

## Status
Accepted

## Context
The platform requires:
- Event ordering
- Duplicate protection
- Managed infrastructure

## Decision
Use AWS SQS FIFO.

## Consequences
Advantages:
- Ordering guarantees
- Reduced operational complexity
Trade-offs:
- Lower throughput than standard queues