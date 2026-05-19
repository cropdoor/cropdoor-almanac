# Persistence and transactions

PostgreSQL 18 is the system of record. Schema lives entirely in `src/main/resources/db/migration/` as Flyway `V<n>__description.sql` files. Redis is for ephemeral state only — nothing durable lives there.

## Flyway: the two hard rules

### Never edit an applied migration

Flyway computes a checksum on every file in `flyway_schema_history`. Editing a file after it has run anywhere — local, testing, staging, prod — produces a checksum mismatch and the application refuses to start (`spring.flyway.validate-on-migrate=true`).

Fix forward with a new `V<n+1>` migration. Always. Even if the original migration was wrong, even if no production data is yet affected, even if it's only ten minutes old. The moment a migration runs anywhere, it's frozen.

### Don't write `${...}` in migration files unless you mean it as a placeholder

Flyway's `PlaceholderReplacingReader` scans the **entire file** — including SQL comments and string literals — before the SQL parser runs. An unregistered `${TOKEN}` aborts the migration and unwinds the entire application context (Flyway → EntityManagerFactory → repositories → filters → Tomcat all collapse).

If you need to document a placeholder shape in a comment, write it in prose form:

- `[CDN_BASE_URL]/[s3_key]` — fine
- `<CDN base URL> + "/" + <s3 key>` — fine
- `${CDN_BASE_URL}/${s3_key}` — fatal

When you do introduce a real placeholder, register a value in `application.properties` under `spring.flyway.placeholders.<name>=...` and override per-profile.

## JSONB usage

JSONB is used **deliberately, not as an escape hatch**. There is exactly one routine JSONB column in the schema: `audit_log.details`. It carries structured key-value pairs whose shape is owned by `AuditEmitter` and consumed by `OrgAuditViewService` via `function('jsonb_extract_path_text', a.details, 'ownerType')`.

Adding JSONB columns elsewhere should be a deliberate architectural decision, not a way to defer normalization. If you're tempted, write a decision record first.

## Transactional policy

- **Services own `@Transactional` boundaries.** A single service method is one unit of work. Repositories don't declare their own transactions; they inherit the caller's.
- **No `@Transactional` on controllers.** Controllers do parameter binding and DTO conversion. Anything transactional gets pushed down into a service call.
- **`@Transactional(readOnly = true)` for pure reads.** Lets Hibernate skip dirty-checking and gives the planner an honest signal. Routine on `*ViewService` and on read-only repositories' callers.

## Async work and the listener pattern

CropDoor has two flavors of work that happen *after* a request returns:

### `@Async` task executor

Configured in `AsyncConfig` — a `ThreadPoolTaskExecutor` with a sized core + max + queue. Used by both audit listeners and email listeners. Runs off the request thread, so the response goes out without waiting on side effects.

### Transactional event listeners

The canonical example is `AuditEventListener`:

```java
@Async
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION, fallbackExecution = true)
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void handleAuditEvent(AuditEvent event) { ... }
```

Three annotations doing three distinct jobs:

| Annotation | Why it matters |
| --- | --- |
| `AFTER_COMPLETION` | Fires whether the outer transaction committed or rolled back. With `AFTER_COMMIT`, failed-login audits silently disappeared because the outer tx always rolls back on auth failure. |
| `REQUIRES_NEW` | Opens its own transaction. The outer one is already closed by the time the listener runs. |
| `@Async` | Runs on the task executor, off the request thread. |

**Why this matters:** failed-login audits are emitted from a `catch (BadCredentialsException) { auditEmitter.loginFailed(...); throw; }` block. The throw rolls back the outer transaction. If audit persistence was tied to the outer tx, every failed login would silently disappear from the record. With `AFTER_COMPLETION` + `REQUIRES_NEW`, it persists regardless.

Email events use the same pattern for the same reason — published in-transaction, sent out-of-transaction, so a transactional failure doesn't leak partial-state emails.

## What doesn't go through this pipeline

- **Schema changes** — Flyway only. Never `hibernate.hbm2ddl.auto=update`. Never raw `ALTER TABLE` from a service.
- **Cross-aggregate reads in a single transaction** — fine. JPA's persistence context is request-scoped via OSIV being explicitly off.
- **N+1-prone queries** — should be `@EntityGraph` or explicit fetch-joined JPQL. The repository declares the fetch shape; the service trusts it.

## Related

- [Request lifecycle](request-lifecycle.md) — where the transaction boundary sits in the request flow
- [Audit logging](../audit/index.md) — full async-listener pipeline
- [ArchUnit pins](archunit-pins.md) — invariants that pin some of these rules at build time
