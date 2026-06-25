# AGENTS.md — Engineering Hub

> Single source of truth for agent workflow, constraints, and context routing.
> Last updated: 2026-06-25 | Architecture: Hexagonal + Spring Boot microservices

---

## Purpose

This repo is a **Spring Boot AI Engineering Playbook** for building production-grade
microservices: `order-service`, `payment-service`, `notification-service`, `inventory-service`.

---

## Engineering Principles (hard rules — non-negotiable)

- **Database-per-service** — no service reads another's DB directly
- **Constructor injection only** — never `@Autowired` on fields
- **Hexagonal architecture** — domain has zero framework dependencies
- **Immutable value objects** — use `record` for DTOs and domain events
- **Events are contracts** — never change published event schema; add fields only
- **Persistence = Spring Data JPA** — see ADR-002; no MyBatis

---

## Workflow (follow for every task)

1. Identify task type → run Context Routing table below
2. Read only the docs listed for that task type
3. Check ADRs for any decision that affects your change
4. Plan in comments before writing code
5. Implement in domain layer first, then adapter
6. Write/update unit tests alongside code
7. Verify checklist in the relevant `.mdc` rule before finalizing

---

## Context Routing

> **Do NOT read all docs for every task. Load only rows matching your task.**

| Task type | Read first |
|-----------|------------|
| Order business logic | `docs/business/order-domain.md` |
| Payment business logic | `docs/business/payment-domain.md` |
| New REST endpoint | `docs/architecture/backend-architecture.md` |
| Database / migration | `docs/database/persistence.md` + `docs/database/migration-strategy.md` |
| Event producer / consumer | `docs/architecture/event-driven-architecture.md` + `docs/adr/ADR-001.md` |
| Cross-service change | `docs/architecture/service-boundaries.md` |
| AWS / infra / deploy | `docs/aws/` |
| Security concern | `docs/architecture/security.md` |
| Architecture decision needed | `docs/adr/` (all) |

---

## Hard Constraints

**NEVER:**
- Read data directly from another service's database
- Use `@Autowired` on fields
- Put business logic in `@RestController` or `@Service` (adapter layer)
- Deploy without migration scripts for schema changes
- Change published event field names or types (backward-incompatible)
- Use `MyBatis` — JPA only (ADR-002)

**ALWAYS:**
- Validate at domain layer, not controller
- Use `record` for DTOs
- Follow naming: `{domain}Created`, `{domain}Updated`, `{domain}Failed` for events
- Write migration SQL in `src/main/resources/db/migration/`

---

## Conflict Resolution

If docs appear to contradict each other:
1. ADRs win over all other docs
2. `.mdc` rules win over `.md` docs for implementation detail
3. Open a PR comment tagging `@team` before proceeding
