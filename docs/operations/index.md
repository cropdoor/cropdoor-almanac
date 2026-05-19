# Operations

Running CropDoor — locally, in CI, and across the four environments.

The promotion flow is `develop` → `testing` (`nursery`) → `staging` (`harvest`) → `production`. Every environment is a Docker image built by GitHub Actions and deployed via Dokploy.

## Planned pages

- **Local development** — Java 21, Docker + Compose, the Maven wrapper, the Makefile entry points, install-hooks script.
- **Environment variables** — the full inventory: DB, Redis, JWT, Resend, Arkesel, Google OAuth, S3 + CDN, bootstrap admin seed, MFA, rate limit thresholds.
- **Configuration profiles** — `application.properties`, `application-test.properties`, profile-specific overrides, the `spring.flyway.placeholders.*` rule (Flyway scans the entire file for `${...}`).
- **CI pipeline** — `ci.yml` (spotless + verify, Postgres + Redis service containers).
- **Deploy pipeline** — `deploy.yml`, the four-branch model, why `testing` skips the verify gate.
- **Observability** — `MetricsService`, the audit-emission failure counter, correlation IDs in logs, planned dashboards.
- **Operational runbooks** — db backup/restore, super-admin seeding, secret rotation (JWT, webhook), bootstrapping the unverified-user purge cron.
- **Test data** — the shared testing-env personas and how the `TEST_SEED_REFRESHED` endpoint works.

!!! info "Status"
    Scaffolding. Source material in `cropdoor-backend/Makefile`, `.github/workflows/`, `docker-compose.yaml`, `Dockerfile`, `application.properties`, and `docs/test-data.md`.
