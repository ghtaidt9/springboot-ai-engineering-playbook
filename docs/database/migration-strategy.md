# Database Migration Strategy

Tool:

Flyway

Rules:

- Forward only migrations
- Immutable scripts
- Version naming:

V1__create_order_table.sql

V2__add_refund_table.sql

No manual database changes in production.