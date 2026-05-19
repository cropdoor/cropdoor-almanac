# RBAC & Permissions

How CropDoor decides who can do what.

The model is **two-tier**: platform-admin RBAC (managed by `SUPER_ADMIN`) sits parallel to org-scoped RBAC (managed by each farm's or buyer's `Owner`). Every permission is a code in `<SCOPE>::<DOMAIN>::<ACTION>` format, every check funnels through a `Permissions.*` constant, and every constant has a matching DB row. ArchUnit pins this consistency.

## Planned pages

- **The two-tier model** — PLATFORM vs FARM vs BUYER, one-org-per-user constraint, the `STAFF_DORMANT_NO_MEMBERSHIP` case.
- **Permission catalog** — the full list grouped by scope, with the code-naming convention.
- **System roles** — `SUPER_ADMIN`, `Owner` (immutable + non-invitable).
- **Custom roles** — creating, updating, deleting org roles; the no-privilege-escalation rule.
- **Permission resolution** — `PermissionResolutionService.effectivePermissionCodes(user)`, how role-permission joins resolve, the candidate caching strategy.
- **Three-layer authorization** — `@PreAuthorize` (admission) → `requireActiveMembership` (org boundary) → `enforceNoEscalation` (write-time guard). Why three layers, not one.
- **Hard invariants** — last super-admin protection, no self-modification, PLATFORM-roles-must-MFA, Owner immutability.
- **Recipe — adding a new permission** — pick code, add `Permissions.X` constant, Flyway migration, wire to endpoint, update slice tests.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/security/Permissions.java`, `service/rbac/`, `model/rbac/`. ArchUnit pin: `PermissionsCatalogConsistencyTest`.
