# Review Checklist — Layered Spring Boot

Review all files just generated for the feature. For each item below, respond:
- ✅ pass — with the file + line reference
- ❌ fail — with exact issue and suggested fix
- N/A — not applicable to this feature

Do NOT summarize. Go item by item.

---

## Entity & Migration

- [ ] `@Entity` has no business logic or state transition methods
- [ ] `@Entity` has explicit `@Table(name = "...")` 
- [ ] Primary key uses `@UuidGenerator`
- [ ] Audit fields present: `createdAt`, `updatedAt` via `@EntityListeners`
- [ ] Flyway migration file exists for every schema change
- [ ] Migration filename format: `V{3-digit}__{snake_description}.sql`
- [ ] Migration has indexes for foreign keys and frequently queried columns

## Repository

- [ ] Interface extends `JpaRepository` — no business logic inside
- [ ] Custom queries use JPQL, not `nativeQuery = true`
- [ ] No `@Transactional` on repository methods

## Service

- [ ] `@Transactional` present on every write method
- [ ] Business rules validated before `repository.save()`
- [ ] `repository.save()` called before `eventPublisher.publish()`
- [ ] Returns DTO, never `@Entity`, to caller
- [ ] Constructor injection only — no `@Autowired` on fields
- [ ] All edge cases from acceptance criteria handled

## Controller

- [ ] Injects `@Service`, never `@Repository`
- [ ] Zero business logic — only HTTP parsing + delegate to service
- [ ] `@Valid` present on every `@RequestBody`
- [ ] POST returns 201 with `Location` header
- [ ] DELETE returns 204
- [ ] `@Operation` and `@ApiResponse` OpenAPI annotations present

## DTOs

- [ ] All request/response types are `record`
- [ ] Request fields have Bean Validation annotations (`@NotBlank`, `@NotEmpty`, etc.)
- [ ] Service never passes `@Entity` where `record` DTO is expected

## Events (if applicable)

- [ ] Event envelope has all 7 fields: `eventId`, `eventType`, `aggregateId`, `aggregateType`, `occurredAt`, `correlationId`, `payload`
- [ ] Publisher called from `@Service` after `save()`, not from Controller
- [ ] `messageGroupId` = `aggregateId`
- [ ] `messageDeduplicationId` = `eventId`
- [ ] Consumer delegates to `@Service` — no inline business logic in listener

## Mapper

- [ ] MapStruct `@Mapper(componentModel = "spring")` present
- [ ] `@Mapping(target = "id", ignore = true)` on toEntity mapping
- [ ] No manual mapping code — use MapStruct only

## Tests

- [ ] `OrderServiceTest` exists — uses `@ExtendWith(MockitoExtension.class)`
- [ ] Happy path tested in service test
- [ ] At least one failure/validation path tested in service test
- [ ] `OrderControllerTest` exists — uses `@WebMvcTest`
- [ ] Controller test mocks `@Service`, not `@Repository`
- [ ] Validation failure returns 400 with standard error format tested
- [ ] No `Thread.sleep()` in tests

## General

- [ ] No `TODO` or `FIXME` left in generated code
- [ ] No cross-service `@Repository` injection
- [ ] Package structure matches: `controller/`, `service/`, `repository/`, `entity/`, `dto/`, `mapper/`, `exception/`

---

## Output format

List every ❌ first with file path, line number, and one-line fix.
Then list all ✅.
Then state: READY TO TEST or NEEDS FIX before testing.
