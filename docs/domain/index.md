# Domain

The business model: farms, buyers, members, listings, orders, and the payment + commission layer that sits underneath them.

CropDoor is a two-sided marketplace. Farms list produce; buyers place orders. Both sides have their own org-scoped role catalog. Orders run through a state machine (PENDING → ACCEPTED → PROCESSING → READY_FOR_PICKUP → IN_TRANSIT → DELIVERED, plus CANCELLED) with two-sided audits and immutable status history.

## Planned pages

- **Farms** — entity model, creation flow (auto-mint Owner role + Member), soft-delete + restore window, the `FARM_DELETED_BY_*` / `FARM_UNDELETED_BY_ADMIN` audits.
- **Buyer profiles** — parallel to farms; the buyer-side `Owner` system role.
- **Members + invites** — `Member` lifecycle (`PENDING` → `ACTIVE` → `REVOKED` / `EXPIRED`), the partial unique index that pins one-org-per-user, the invite/accept/expire/purge cron, re-invite race fixes.
- **Listings + image storage** — `Listing`, `ListingImage`, the S3 + CDN pipeline, the per-listing image cap (currently 4 via env override), the `S3ListingImageStorage` fail-fast guard on `cropdoor.cdn.base-url`.
- **Order lifecycle** — the full state machine, `OrderStatusHistory`, two-sided audit emission, `CancellationSide`, on-time delivery computation.
- **Order taxes and fees** — `OrderTax`, `OrderCommission`, `Fee`, `CommissionRate`. How per-order totals are computed.
- **Payments + payouts** — `Payment`, `PaymentAttempt`, `Payout`. **Note:** the model exists; provider integration (escrow, settlement) is on the roadmap, not in code. This page will clearly mark current-vs-planned.
- **Disputes + reviews** — `Dispute`, `Review`. Roadmap surface.
- **Drivers + delivery** — `DriverProfile`, `Delivery`. Roadmap surface.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/model/` and parallel `service/` packages.
