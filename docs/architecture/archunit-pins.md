# ArchUnit pins

Architecture is documented; load-bearing rules are also **pinned at build time** via ArchUnit tests under `src/test/java/com/cropdoor/backend/architecture/`.

A pin is the difference between a convention (documented, easy to drift) and an invariant (enforced, fails CI). Every rule in the table below is enforced — break one, CI fails before merge.

## The pins

| Test | What it enforces |
| --- | --- |
| `AuditAopArchitectureTest` | `@Audited` has exactly one method named `value`. `@AuditPrincipal` is a zero-method marker. Every `@Audited` method is public, non-static, non-final. `@AuditPrincipal` only marks `User`-typed parameters. The `aspect/` directory stays under its 6-file cap. |
| `OrgIsolationArchitectureTest` | No class in `controller/farm/**` injects buyer-side beans, and vice versa. No class in `service/farm/**` injects `BuyerProfileService`, and vice versa. Cross-side leakage fails CI. |
| `PermissionsCatalogConsistencyTest` | Every `Permissions.*` constant matches the `<SCOPE>::<DOMAIN>::<ACTION>` pattern. Allowed scopes and actions are pinned to an allow-list. Constant names mirror their code values (`::` → `_`). |
| `EmailEventListenerArchitectureTest` | Pin against direct Resend send-calls; emails must flow through the event-listener pipeline. |

## Why pin these specifically

Each one guards against a real failure mode we've seen — or expect to see — that human-readable docs alone can't prevent:

### Audit AOP shape (`AuditAopArchitectureTest`)

The `@Audited` annotation is intentionally small (one `AuditAction` value, nothing else). Multiple times in code review someone proposed adding `details`, `entityType`, or SpEL parameters. The pin makes that impossible without first removing the test — which is a visible signal.

The 6-file cap on `aspect/` is a complexity budget. Aspects are powerful and opaque; adding a seventh one should require a design conversation, not a quiet PR.

### Org isolation (`OrgIsolationArchitectureTest`)

Farms and buyers each have their own org-scoped role catalog, member lifecycle, and permission set. They're parallel, not sibling. The pin prevents the obvious failure mode where a farm controller starts accepting buyer-side context, or vice versa — a class of cross-tenant IDOR bug that's hard to see until production.

### Permission catalog consistency (`PermissionsCatalogConsistencyTest`)

Every permission code goes through a `Permissions.*` constant. Every constant has a matching DB row. The pin verifies the constant name shape (`FARM_LISTING_CREATE` for code `"FARM::LISTING::CREATE"`) and the allowed scope/action vocabulary. Drift here would mean code references permissions that aren't in the database — silent 403s or worse, silently-granted access.

### Email pipeline (`EmailEventListenerArchitectureTest`)

Emails always flow through `event/email/*` listeners. The pin prevents anyone from calling `ResendClient.send(...)` directly from a service — that bypasses the async pipeline, leaks partial-state emails on transaction rollback, and makes testing harder. The pin makes the right path the only path.

## When to update a pin

You should update a pin when the underlying architectural rule legitimately changes — not when you want a feature that conflicts with the pin.

If you propose to remove or weaken a pin in a PR, expect that PR to slow down: the discussion is whether the *rule* should change, not whether your feature deserves an exception.

If you add a new architectural rule that should be invariant (not just convention), add a new ArchUnit test alongside the docs that describe it.

## Related

- [Module map](module-map.md) — packages that the pins target
- [Audit logging](../audit/index.md) — the AOP surface that `AuditAopArchitectureTest` guards
- [RBAC & Permissions](../rbac/index.md) — catalog rules that `PermissionsCatalogConsistencyTest` guards
