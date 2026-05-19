# Module map

Every top-level Java package under `com.cropdoor.backend.*` and what it's responsible for.

The naming convention is **one package per bounded context**, with the same name reused across `controller`, `dto`, `mapper`, `model`, `repository`, and `service`. A new domain area gets six folders, not one.

## Package table

| Package | Owns |
| --- | --- |
| `annotations` | Custom Java annotations |
| `aspect/audit` | `@Audited` annotation, the audit AOP aspect, principal resolution |
| `aspect/validation` | Boot-time validation post-processors (e.g. `AuditedMethodValidationPostProcessor`) |
| `bootstrap` | Startup runners (super-admin seeding, waitlist backfill) |
| `config` | `@Configuration` beans — CORS, Resend, Async, Clock, JPA auditing, S3, Swagger |
| `controller/{admin,auth,buyer,farm,order,waitlist,webhook}` | REST controllers grouped by domain side |
| `dto/{domain}/{request,response}` | Request + response records (Java records, immutable) |
| `event` | Spring application events (`AuditEvent`, email events) and their listeners |
| `exception` | Custom exceptions and `GlobalExceptionHandler` |
| `job` | Scheduled jobs (unverified-user purge cron, invite expiry sweeps) |
| `mapper` | MapStruct interfaces; one hand-written mapper where business logic intrudes |
| `model/{address,buyer,commission,common,delivery,dispute,driver,farm,fee,listing,order,payment,platform,rbac,review,user,waitlist}` | JPA entities, grouped by aggregate |
| `observability` | `MetricsService`, health indicators |
| `repository/{domain}` | Spring Data JPA repositories |
| `security` | Spring Security config, JWT, OAuth, rate limiting, correlation ID, password policy |
| `service/{admin,auth,buyer,farm,listing,notification,order,org,platform,rbac,waitlist}` | Business logic — one service package per bounded context |
| `util` | Pure helpers (E.164 normalization, hashing, etc.) |
| `validation` | Bean Validation constraints + validators |

## Why six folders per domain

When a new bounded context arrives — say, `disputes` — the convention is to create:

- `controller/dispute/` — REST surface
- `dto/dispute/{request,response}/` — wire DTOs
- `mapper/dispute/` — entity ↔ DTO mappers
- `model/dispute/` — JPA entities and value types
- `repository/dispute/` — Spring Data interfaces
- `service/dispute/` — business logic

This sounds like ceremony, but it pays off the second time a bounded context grows: searching for "everything related to disputes" is one folder glob away in every concern.

## What lives outside this map

A few things aren't in the package list above because they're not under `com.cropdoor.backend.*` at all:

- **`src/main/resources/db/migration/`** — Flyway SQL migrations. The system of record for schema. See [Persistence and migrations](persistence.md).
- **`src/main/resources/application.properties` (and profiles)** — runtime configuration. Environment-specific overrides live in `application-{test,prod}.properties`.
- **`src/test/java/com/cropdoor/backend/architecture/`** — the ArchUnit pinning tests that enforce the load-bearing invariants. See [ArchUnit pins](archunit-pins.md).

## Related

- [Request lifecycle](request-lifecycle.md) — how a request crosses these packages
- [Persistence and migrations](persistence.md) — schema layer that sits below `repository/`
- [ArchUnit pins](archunit-pins.md) — invariants enforced across packages
