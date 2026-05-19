# Roadmap surfaces

Entities and packages that exist in `model/` but aren't yet wired through the service + controller + audit pipeline. Listed here so contributors don't mistake "the entity exists" for "the feature ships."

Every page in the almanac that touches one of these areas marks current-vs-planned explicitly at the top — never leave a reader guessing.

## In model, not in flow

| Surface | What's in code | What's not |
| --- | --- | --- |
| **Payments** | `Payment`, `PaymentAttempt`, `Payout`, `PaymentStatus`, `PayoutStatus` entities + Flyway schema | Provider integration (escrow PSP, settlement, webhooks). No service calls a real payment API. Order placement does not charge. Order delivery does not settle. |
| **Disputes** | `Dispute` entity | Dispute lifecycle service, controller endpoints, audit actions, admin resolution UI. |
| **Drivers + delivery** | `DriverProfile`, `Delivery` entities | Driver onboarding flow, delivery assignment, `DELIVERY_*` audit actions, on-time computation against the order. |
| **Reviews** | `Review` entity | Buyer-leaves-review-for-farm flow, farmer responses, moderation. |

The full picture for each lives on its domain page: see [Payments](../payments/index.md), and (when written) the Domain section's Disputes / Drivers / Reviews subpages.

## In flow, not roadmap

A separate category — these *look* like roadmap from the entity count but are actually shipped:

- **Google OAuth** — `OAuthRoleStateStore`, `RoleAwareOAuth2AuthorizationRequestResolver`, the success + failure handlers, `OAUTH_SIGNUP` and `OAUTH_LINKED` audits are all in code and wired. See [Security](../security/index.md).
- **Phone-verification gate** — currently in flow on the `phone-verification-gate` feature branch. `VerificationChallengeResponse`, OTP channel routing, the unverified-user purge cron. Will be folded into [Authentication](../auth/index.md) once it merges to `main`.
- **Listings + image storage** — `S3ListingImageStorage`, `ListingImageStorageProperties`, CDN base-URL validation, presigned-upload flow. Image cap is enforced via the `cropdoor.listings.images.max-per-listing` property (currently 4).

## What "roadmap" means in CropDoor's context

A roadmap surface is **scaffolded but not load-bearing**. The schema exists so adding the feature later doesn't require a migration; the entity exists so foreign keys can already point at it; but no production code path produces or consumes it yet.

This is deliberate. It's much cheaper to add Flyway migrations early (when the schema isn't yet on prod) than later (when migrations become irreversible). It's much cheaper to define an entity early than to retro-fit `FK references` after orders + payments are entwined.

The risk is mistaking presence for completeness. If you grep `model/payment/` and see four classes, you might assume payments work. They don't. The almanac's job is to make that gap visible.

## How to remove a surface from this list

When a roadmap surface gets fully wired (service + controller + audit actions + migrations + tests), move it off this page and into its proper section's main page. Update the section's status admonition to drop the "Mostly roadmap today" warning.

The list above shrinking over time is a useful health signal for the project.

## Related

- [Module map](module-map.md) — where the roadmap entities live in the package layout
- [Payments](../payments/index.md) — the most-discussed roadmap surface
