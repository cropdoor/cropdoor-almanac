# Authentication

Registration, login, OTP, MFA, and the phone-verification gate.

CropDoor has multiple identity surfaces — regular users, platform admins, organisation members — and several login channels (password, OAuth, phone OTP). Every successful path returns the same `AuthResponse` shape, but the routes to get there differ.

## Planned pages

- **Principal model** — `User`, `PlatformAdmin`, `Member`. Why they're three separate entities and stay separate.
- **Registration** — standard signup, waitlist-claim signup, the `REGISTER` vs `REGISTER_RESUMED` distinction, the unverified-user purge cron.
- **Standard login** — password authentication, the failure branches (`LOGIN_FAILED` reasons), gate-phase vs post-auth failures.
- **Phone-verification gate** — the `phone-verification-gate` branch arc: `VerificationChallengeResponse`, OTP channel routing, `OTP_VERIFIED` minting tokens on first verification, `LOGIN_VERIFICATION_REQUIRED`.
- **Admin login + MFA** — two-phase login, `ADMIN_LOGIN` with `stage=mfa-pending`, MFA challenge resolution.
- **OAuth (Google)** — login vs signup, post-OAuth phone verification at `POST /v1/auth/oauth/signup/complete`.
- **Refresh-token rotation** — token rotation policy, `REFRESH_TOKEN_REUSE_DETECTED` and what triggers it.
- **Logout** — single-device vs `scope=all_devices`.
- **Forgot password / reset password** — SMS vs email flow.
- **Email verification** — verification token lifecycle.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/service/auth/` and `controller/auth/`. The existing `cropdoor-backend/docs/auth/auth-flows.md` is a reference but pre-dates the phone-verification-gate work.
