# Event Driven Architecture

## Messaging Technology
AWS SQS FIFO
AWS SNS

## Design Principles
Events must be:
- Immutable
- Versioned
- Idempotent
Consumers must:
- Handle duplicates
- Support retries
- Implement dead-letter handling
