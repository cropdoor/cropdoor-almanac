# Domain

The business model: farms, buyers, members, listings, orders, and the payment + commission layer that sits underneath them.

CropDoor is a two-sided marketplace. Farms list produce; buyers place orders. Both sides have their own org-scoped role catalog. Orders run through one lifecycle (AWAITING_PAYMENT → PENDING → ACCEPTED → PROCESSING → READY_FOR_PICKUP → IN_TRANSIT → DELIVERED, plus CANCELLED) with two-sided audits. The flow is drawn end to end, with the rebuild under deliberation, in [The order → delivery flow](order-delivery-flow.md) — and in plain language for Operations in [From farm gate to front door](order-delivery-flow-for-operations.md).

## Planned pages

- **Farms** — entity model, creation flow (auto-mint Owner role + Member), soft-delete + restore window, the `FARM_DELETED_BY_*` / `FARM_UNDELETED_BY_ADMIN` audits.
- **Buyer profiles** — parallel to farms; the buyer-side `Owner` system role.
- **Members + invites** — `Member` lifecycle (`PENDING` → `ACTIVE` → `REVOKED` / `EXPIRED`), the partial unique index that pins one-org-per-user, the invite/accept/expire/purge cron, re-invite race fixes.
- **Listings + image storage** — `Listing`, `ListingImage`, the S3 + CDN pipeline, the per-listing image cap (currently 4 via env override), the `S3ListingImageStorage` fail-fast guard on `cropdoor.cdn.base-url`.
- **Order lifecycle** — see [The order → delivery flow](order-delivery-flow.md): states, actors, side effects, money, and the proposed deletions (including `OrderStatusHistory`, which nothing reads).
- **Order taxes and fees** — `OrderTax`, `OrderCommission`, `Fee`, `CommissionRate`. How per-order totals are computed.
- **Payments + payouts** — `Payment`, `PaymentAttempt`, `Payout`. Shipped: Paystack escrow checkout, payouts, refunds, chargebacks, and the double-entry ledger. The full picture lives in the [Payments](../payments/index.md) section.
- **Reviews + in-app disputes** — `Review`, `model/dispute/*`. Roadmap surface. *(Not to be confused with Paystack chargeback dispute-defense, which is shipped — see [Payments](../payments/index.md).)*
- **Drivers + delivery** — `DriverProfile`, `DeliveryRun`, agent pickup and confirmation. Shipped; drawn in [The order → delivery flow](order-delivery-flow.md). `Delivery` itself is proposed for removal there.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/model/` and parallel `service/` packages.
