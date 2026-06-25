# PostgreSQL Guidelines

## Naming Convention
Tables:
snake_case
Columns:
snake_case
Primary Keys:
UUID

## Indexing
Indexes required for:
- Foreign keys
- Search fields
- High traffic queries

## Ownership
Each service owns its own schema.