# Payments

The financial layer underneath the order lifecycle: payments in (from buyers), payouts out (to farms), and the commission + fee + tax accounting that sits between the two.

!!! warning "Mostly roadmap surface"
    Payment + payout **entities and schema** exist in code today. **Provider integration** (a real PSP for escrow, settlement, payouts, refunds) is not wired. Order placement does not currently charge a card; order delivery does not currently settle to a farm bank account. Every page in this section will mark current-vs-planned at the top.

## What's in code today

| Layer | Where |
| --- | --- |
| `Payment`, `PaymentAttempt`, `PaymentStatus`, `PaymentAttemptStatus` | `model/payment/` |
| `Payout`, `PayoutStatus` | `model/payment/` |
| `CommissionRate` | `model/commission/` |
| `Fee`, `FeeType` | `model/fee/` |
| `OrderCommission`, `OrderTax`, `TaxBasis`, `TaxType` | `model/order/` |
| Schema | Flyway migrations `V7` (fees), `V8` + `V12` (commissions), `V9` (payment attempts), `V13` (order taxes) |

The entities are JPA-mapped and the schema is migrated, but there's no `PaymentService`, no controller endpoint that calls a PSP, and no webhook handler for settlement events. The closest thing to a decision record on this layer is the (now-deleted-from-the-nav) commission-per-order ADR — that one is preserved in git history.

## Planned pages

- **Payment lifecycle** — the `PaymentStatus` state machine, `PaymentAttempt` retry semantics, how an order transitions from PENDING → ACCEPTED without (currently) hitting a PSP.
- **Payouts** — `Payout`, `PayoutStatus`, when a payout is created, how it relates to fulfilled orders, who initiates it (system vs admin).
- **Commissions** — `CommissionRate` per-listing or per-farm, `OrderCommission` rows snapshotted at order placement (the ADR called this "per-order commission" — rates are frozen at order time so historical rate changes don't retroactively rewrite settled orders).
- **Fees** — `Fee`, `FeeType`. Platform fees vs delivery fees, when they're applied, how they show up in the order total.
- **Taxes** — `OrderTax`, `TaxBasis`, `TaxType`. Currently a schema scaffold; tax computation is roadmap.
- **Provider integration (roadmap)** — which PSP, escrow model, webhook surface, idempotency keys, reconciliation flow. Open design question; no decision yet.
- **Reconciliation + payouts ops (roadmap)** — manual ops surface for admins to inspect failed attempts, retry payouts, audit settlement deltas.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/model/{payment,commission,fee}/` and `model/order/OrderCommission.java`, `OrderTax.java`. Schema in `src/main/resources/db/migration/V7`, `V8`, `V9`, `V12`, `V13`.
