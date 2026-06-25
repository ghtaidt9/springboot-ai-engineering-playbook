# Persistence Standards

## ORM

Technology:

Spring Data JPA

Provider:

Hibernate ORM

Database:

PostgreSQL

---

## Entity Rules

Use:

@Entity
@Table

Primary Key:

UUID

Audit Fields:

created_at
updated_at

Never:

Use Lombok @Data

Prefer:

@Getter
@Setter
@NoArgsConstructor