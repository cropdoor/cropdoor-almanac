# Audit Logging

Every security-relevant event in CropDoor produces a row in `audit_log` with a typed action, optional principal user, and a JSON `details` map keyed by the canonical `AuditKeys.KEY_*` constants.

The pipeline has five layers: a typed catalog (`AuditAction`), an explicit wire contract (`AuditKeys`), a single-source-of-truth emitter (`AuditEmitter` / `AuditEmitterImpl`), an async event listener that persists rows independently of the caller's transaction, and a sugar aspect (`@Audited`) for the trivial happy-path subset.

## Planned pages

- **Subsystem overview** — five-layer pipeline diagram, what each layer owns, why the event-bus indirection exists.
- **AuditAction catalog** — every action, when it fires, what makes it distinct from siblings. Grouped by domain (auth, OTP, OAuth, admin, members, org roles, org lifecycle, listings, orders).
- **The details-map contract (`AuditKeys`)** — every canonical key, what it means, the rule that renames break observability.
- **AuditEmitter** — typed-method-per-action design, context records (`LoginContext`, `OrderContext`, …), the rule against polymorphic `emit(action, details)`.
- **Async persistence** — `AuditEvent`, `AuditEventListener`, the `AFTER_COMPLETION` + `REQUIRES_NEW` + `@Async` choice, why `AFTER_COMMIT` would silently drop failed-login audits.
- **AOP path (`@Audited`)** — when to reach for the annotation vs calling the emitter directly, the boot-time validation post-processor, the dispatch table.
- **Per-org audit feed** — `OrgAuditViewService`, the JSONB-extraction query, the visible-action whitelist, the `KEY_OWNER_TYPE` + `KEY_OWNER_ID` rule.
- **Architecture pins** — `AuditAopArchitectureTest`, the aspect-directory 6-file cap.
- **Adding a new audit event** — add to `AuditAction`, add method to `AuditEmitter` + impl, choose direct vs `@Audited`, update the org-feed whitelist if relevant.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/model/platform/`, `service/platform/`, `aspect/audit/`, `event/`, `service/org/`. The existing `cropdoor-backend/docs/audit/audit-subsystem.md` is a 600-line reference; this almanac will absorb it after the drift is fixed. Sub-pages get split out as each topic is documented.
