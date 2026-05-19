---
hide:
  - navigation
  - toc
---

# CropDoor Almanac

The reference book for the CropDoor farm-to-buyer marketplace. Architecture, subsystems, and the load-bearing invariants that hold the platform together.

If you're a new contributor, start with [Architecture overview](architecture/index.md). If you're chasing a specific subsystem, jump straight in.

## Sections

<div class="grid cards" markdown>

- :material-source-branch:{ .lg .middle } **[Architecture](architecture/index.md)**

    ---

    Module map, request lifecycle, the high-level view. Start here.

- :material-shield-lock:{ .lg .middle } **[Security](security/index.md)**

    ---

    JWT, OAuth, rate limiting, CORS, key rotation, password policy.

- :material-account-key:{ .lg .middle } **[Authentication](auth/index.md)**

    ---

    Registration, login, OTP, MFA, phone-verification gate, refresh tokens.

- :material-account-multiple-check:{ .lg .middle } **[RBAC & Permissions](rbac/index.md)**

    ---

    Two-tier model (platform + org), permission catalog, three-layer authorization.

- :material-clipboard-text-clock:{ .lg .middle } **[Audit Logging](audit/index.md)**

    ---

    AuditEmitter, AuditEvent flow, per-org feed, action catalog.

- :material-domain:{ .lg .middle } **[Domain](domain/index.md)**

    ---

    Farms, buyers, members, listings, orders — the business model.

- :material-cash-multiple:{ .lg .middle } **[Payments](payments/index.md)**

    ---

    Payments in, payouts out, commissions, fees, taxes. Mostly roadmap today.

- :material-bell-ring:{ .lg .middle } **[Notifications](notifications/index.md)**

    ---

    Email (Resend), SMS (Arkesel), voice OTP, delivery webhooks.

- :material-cog:{ .lg .middle } **[Operations](operations/index.md)**

    ---

    Env vars, deploy flow, Docker, observability, runbooks.

- :material-pencil-plus:{ .lg .middle } **[Contributing](contributing/index.md)**

    ---

    How to add or edit a doc, style conventions.

</div>

## Where this fits

| Stage | Where |
| ----- | ----- |
| Testing | [nursery.cropdoor.com](https://nursery.cropdoor.com) |
| Staging | [field.cropdoor.com](https://field.cropdoor.com) |
| Reference docs | **you are here** |

The almanac is the trusted reference book on the farm. Check it before you change a load-bearing subsystem.
