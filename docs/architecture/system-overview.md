# System Overview

## Architectural Style
The platform follows:
- Microservices Architecture
- Domain Driven Design
- Hexagonal Architecture
- Event Driven Architecture

## Communication
### Synchronous
REST APIs
### Asynchronous
SQS FIFO
SNS

## Data Ownership
Every service owns:
- Its own database
- Its own domain logic
- Its own migrations
Cross-database access is forbidden.