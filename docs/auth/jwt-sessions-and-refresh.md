# JWT, sessions & refresh

This page owns the token machinery behind every authenticated CropDoor request: how an access token and a refresh token are minted and signed, how signing keys rotate without logging anyone out, how the filter chain re-authenticates each request, and how refresh-token rotation detects a stolen token and burns the whole session. Everything above the login flows — [login and the phone gate](login-and-the-phone-gate.md), [admin login and MFA](admin-login-and-mfa.md), [OAuth](oauth.md) — ends by calling `JwtProvider` to produce the pair described here, so this is the shared plumbing they all sit on.

Two facts anchor the rest of the page. First, **authorities are never in the token** — the JWT carries identity and a login-`scope`, and permissions are re-resolved from the database on every single request. Second, **only hashes hit disk** — the raw refresh JWT is returned to the client once and never persisted, and the blacklist stores a SHA-256 digest, not the token. All response payloads below are the `data` field of the `ApiResponse<T>` wrapper.

!!! info "Status"
    The parsing, the filter, the refresh and the logout below are as the security fix *Pending invite sign-in* part 2 leaves them (#379, ready 26 Oct 2026, not yet merged); the places it changed are marked as part 2's. Develop is not promoted beyond testing until it has merged and passed its live verify.

## The token pair — `JwtProvider`

`security/jwt/JwtProvider.java` mints both tokens, signing each with HMAC-SHA512 (`HS512`) using the active signing key and stamping the key's `kid` into the JWS header. `JwtProvider#generateToken` builds the access token; `JwtProvider#generateRefreshToken` builds the refresh token. Both take a `CustomUserDetails` and the resolved `LoginContextResponse`, so the token records *which* context the session runs in.

| Claim | Access token | Refresh token | Meaning |
| --- | --- | --- | --- |
| `kid` (header) | yes | yes | Which signing key verifies this token |
| `sub` | yes | yes | Login identifier — email when present, otherwise phone (`CustomUserDetails#getUsername`) |
| `jti` | yes | yes | Random `UUID` per token |
| `iat` / `exp` | yes | yes | Issued-at / expiry, from the injected `Clock` |
| `userId` | yes | yes | The `users` row id |
| `email` | yes | no | Same value as `sub` (email-or-phone); named `email` for historical reasons |
| `type` | no | `"refresh"` | Marks the refresh token; each door checks it (see the parsing below): the refresh door accepts only `"refresh"`, the access door refuses it |
| `scope` | yes | yes | The `LoginContextType` name, e.g. `PLATFORM_ADMIN`, `FARM_MEMBER`, `BUYER_MEMBER`, `DORMANT` |
| `ownerId` / `ownerType` | org contexts only | org contexts only | The owning farm/buyer org for member contexts |

There are **no role or permission claims**. `JwtProvider#generateToken` documents this deliberately: embedding authorities would leak permission codes in any captured token, bloat the token, and go stale the instant a role is edited. Authorization is instead rebuilt per request (next section) and is the canonical topic of [RBAC authorization](../rbac/authorization.md).

### Parsing: one parse, one shape per door

Since part 2 (#379), every token is parsed and verified in one place, `JwtProvider#verifiedClaims`: the key its `kid` names, the signature, and an `exp` not yet passed. The door's shape check then reads the verified claims in the same step. Each door has exactly one way in, and each refuses the other's token (deny-by-default):

| Method | Read by | Accepts only | Returns |
| --- | --- | --- | --- |
| `JwtProvider#parseAccessToken` | `JwtAuthFilter` on HTTP and the chat WebSocket's `CONNECT` alike | a non-blank `email` and `scope` claim, an `exp`, and no `type` of `refresh` | the `email` claim (the principal is loaded by it) and the `exp` |
| `JwtProvider#parseRefreshToken` | `AuthServiceImpl#refreshToken` | a `type` of `refresh`, a non-blank `sub` and an `exp` | the `sub` |

A token that does not verify or has the wrong shape comes back empty, a claim of the wrong type (a number where a string belongs) included, so it is refused rather than answered with a 500. The old `validateToken` and per-claim readers are gone.

### Token lifetimes

TTLs are role-aware. Non-admins use the platform defaults bound under `app.jwt.*` on `JwtKeyConfig`; admins use the tighter `app.security.admin.*` values on `AdminSessionProperties`, which `JwtProvider` reads directly as the authoritative source for admin token lifetimes.

| Principal | Access TTL | Refresh TTL | Source |
| --- | --- | --- | --- |
| Non-admin (`FARMER` / `BUYER` / `STAFF`) | 15 min (`900000` ms) | 7 days (`604800000` ms) | `JwtKeyConfig` (`app.jwt.expiration-ms`, `app.jwt.refresh-expiration-ms`) |
| Admin (`UserType.ADMIN`) | 5 min (`300000` ms) | 1 day (`86400000` ms) | `AdminSessionProperties` (`app.security.admin.access-ttl-ms`, `app.security.admin.refresh-ttl-ms`) |

One subtlety: the persisted `refresh_tokens.expires_at` row is always stamped with the platform default (7 days) in `RefreshTokenServiceImpl`, even for admins, whereas the admin refresh JWT's own `exp` is 1 day. Because `AuthServiceImpl#refreshToken` accepts the token through `JwtProvider#parseRefreshToken` (which checks the signature and `exp`) before the stored row's `expires_at` is ever consulted, the shorter 1-day JWT expiry governs for admins — the longer DB expiry never gets a chance to matter.

## Signing keys & rotation — `JwtKeyProvider`

Keys are configured as a list under `app.jwt.keys[n]`, each a `JwtKeyEntry` with `kid`, `secret`, and `active`. `security/config/JwtKeyConfig.java` binds the list; `security/jwt/JwtKeyProvider.java` loads it at startup (`@PostConstruct init`) into a `kid → SecretKey` map and enforces the invariants — boot fails fast if:

- no keys are configured,
- any secret is shorter than 64 characters (512 bits, the `HS512` minimum), or
- zero or more-than-one entry is marked `active`.

Exactly one key is the **active signing key** (`JwtKeyProvider#getActiveKey`, its id from `getActiveKid`); every configured key — active or retired — remains a **verification key** reachable by `getKey(kid)`. Verification is keyed by the header: `JwtProvider` reads the `kid` out of the JWS header, resolves the matching `SecretKey`, and verifies with it. A token whose `kid` is missing or unknown fails validation outright.

That split is what makes rotation seamless. New tokens sign with the new active key; tokens already in the wild keep verifying against whichever retired key their `kid` names.

The rotation procedure, which invalidates no live token:

1. Add a new `keys[n]` entry: a fresh `kid`, a fresh secret of 64 or more characters, `active = true`.
2. Flip the previous active entry to `active = false`, but keep it in the list.
3. Redeploy: `JwtKeyProvider` now signs with the new `kid` and still verifies the old one.
4. Wait out the longest refresh TTL (7 days), so every token with the old `kid` has expired.
5. Only then remove the retired `keys[n]` entry.

## Per-request authentication — the filter chain

`security/config/SecurityConfig.java` runs a stateless (`SessionCreationPolicy.STATELESS`) filter chain. Three custom filters are registered ahead of Spring's `UsernamePasswordAuthenticationFilter`, in this order:

1. `CorrelationIdFilter` — seeds the MDC with `requestId` first, so every downstream log line correlates.
2. `JwtAuthFilter` — the bearer-token authenticator described below.
3. `AdminPerUserRateLimitFilter` — post-auth per-admin rate limiting (see [admin login and MFA](admin-login-and-mfa.md)).

`security/jwt/JwtAuthFilter.java` (`OncePerRequestFilter`) re-authenticates on **every** request — there is no server session to reuse. Its logic in `authenticateRequest`:

1. Read `Authorization: Bearer <jwt>`. Absent: continue unauthenticated.
2. Accept it through `JwtProvider#parseAccessToken`. Empty (it does not verify, has expired, or is not an access token, a refresh token included), or blacklisted: continue unauthenticated.
3. Load the principal with `CustomUserDetailsService#loadUserByUsername`, by the token's `email` claim. No such account: continue unauthenticated.
4. The account is not ACTIVE (suspended, or a pending invitee): continue unauthenticated, so a suspension takes effect at the next request.
5. Platform staff (the `ROLE_PLATFORM_STAFF` authority, which only an ACTIVE platform membership grants) whose `admin-idle` key is missing: write a 401 and stop the chain.
6. Set the `SecurityContext` with the principal's own authorities.

The load-bearing step is `loadUserByUsername`: `CustomUserDetailsService` re-derives the principal's authorities from the database (`ROLE_ADMIN` / `ROLE_SUPER_ADMIN` plus bare permission codes, or the member's role-permissions) on each call — this is why a revoked permission takes effect on the very next request without re-issuing tokens. The filter adds no authority from the token. Until part 2 (#379) it also added a `SCOPE_<scope>` authority from the `scope` claim. Nothing gated on it, and part 2 removed it: that claim is resolved at sign-in from the live membership, a pending invite included, so an authority derived from it would grant a pending invitee's token whatever came to gate on it. The `scope` claim stays only as part of the access-token shape. Authority resolution is the canonical topic of [RBAC authorization](../rbac/authorization.md).

```mermaid
flowchart TD
    A["Request with Authorization header"] --> B["JwtAuthFilter"]
    B --> C{"parseAccessToken accepts it, and not blacklisted?"}
    C -->|"no"| U["proceed unauthenticated"]
    C -->|"yes"| E["loadUserByUsername by the email claim — rebuild authorities from DB"]
    E --> D{"account found and ACTIVE?"}
    D -->|"no"| U
    D -->|"yes"| F{"platform staff?"}
    F -->|"no"| G["set SecurityContext, the principal's authorities only"]
    F -->|"yes"| H{"admin-idle key present in Redis?"}
    H -->|"no"| I["401 SESSION_IDLE, stop chain"]
    H -->|"yes"| J["refresh idle TTL, set SecurityContext"]
    U --> K["reaches endpoint; @PreAuthorize / entry point decides"]
    G --> L["authenticated"]
    J --> L
```

A benign no-auth case (no token, a bad or expired token, a refresh token, a blacklisted token, an account that is not ACTIVE) does **not** 401 inside the filter — it lets the request proceed unauthenticated so the authorization layer can decide. If the endpoint requires authentication, Spring Security invokes `security/jwt/JwtAuthEntryPoint.java`, which writes a `401` with a deliberately generic `"Unauthorized"` message and `ErrorCode.INVALID_TOKEN`. The concrete Spring Security exception (bad credentials, locked, unknown user) is logged server-side at WARN but never returned, to avoid handing an unauthenticated caller account-enumeration signal.

The admin idle-session gate — the `admin-idle:<userId>` Redis key and its sliding 30-minute TTL — is enforced here but owned by [admin login and MFA](admin-login-and-mfa.md).

## Token blacklist — `JwtBlacklistService`

`security/jwt/JwtBlacklistServiceImpl.java` is a Redis-backed deny-list used to kill an access token before its natural `exp`. It stores the SHA-256 hex digest of the token under `jwt:blacklist:<digest>` with a TTL equal to the token's **remaining** lifetime — so the entry self-expires exactly when the token would have anyway, and Redis never accumulates dead keys. A zero-or-negative remaining TTL is a no-op. `blacklist` increments a metric via `MetricsService#countJwtBlacklist`. `JwtAuthFilter` calls `isBlacklisted` on every request. Blacklisting happens on logout and on refresh (the outgoing access token is blacklisted), both covered below.

## Refresh rotation & reuse detection — `RefreshTokenService`

`POST /v1/auth/refresh-token` (`AuthController#refreshToken` → `AuthServiceImpl#refreshToken`) exchanges a refresh token for a brand-new access **and** refresh token pair. The endpoint is unauthenticated (the refresh token is the credential), and the caller may pass the old — even expired — access token in the `Authorization` header so it can be blacklisted on the way out. The `RefreshTokenRequest` body carries the refresh token itself.

The service accepts the token through `JwtProvider#parseRefreshToken`, loads the principal by its `sub` through `CustomUserDetailsService` and refuses an account that is not ACTIVE, mints the new pair (re-resolving the login context so a context change is picked up), then hands the rotation to `RefreshTokenServiceImpl#rotateRefreshToken`, with the principal's user as the owner the stored row must belong to. Refresh tokens are **one-time-use**. Raw tokens are never stored — `service/auth/RefreshTokenServiceImpl.java` hashes every token with `util/TokenHasher.java` (SHA-256) and matches on the hash.

### The rotation: insert, then a conditional revoke

Since part 2 (#379), `rotateRefreshToken` takes no lock at the read, and decides each presentation in this order:

1. **No row** for the hash (a garbage or never-issued token): refused.
2. **The row is revoked:** a replay, so reuse detection (below).
3. **The row belongs to another account** than the one the token names: refused. Before part 2, such a token got a 200 and a session for the account the token named.
4. **The row's `expires_at` has passed:** refused.
5. Otherwise the rotation **inserts and flushes its new row, then revokes the presented row with a conditional update**, `RefreshTokenRepository#revokeIfNotYetRevoked`, which sets `revoked = true` only where `revoked = false`.
    - **1 row updated:** this presentation won. The new pair is returned.
    - **0, and the row is gone** (the deploy step deleted it meanwhile): refused, and the new row goes with the rollback.
    - **0, and the row is still there:** another presentation or a revoke got there first. The rotation deletes its new row and goes through reuse detection.

So of two simultaneous presentations of one token, **exactly one wins**; the other trips reuse detection, which revokes every refresh token of the account, the one the winner just received included. The web app serialises its refreshes across tabs with `navigator.locks` for this reason, and any client must send one refresh at a time. The insert comes first on purpose: its foreign-key check holds the user row before the rotation holds the presented row, the order the phone change's revoke-all takes them in, so a rotation forms no lock cycle with the phone change, any revoke-all or the deploy step's table lock.

**Reuse detection** is the theft countermeasure. A rotated row is marked `revoked`, not deleted (it lingers until the cleanup or its expiry), so replaying an already-rotated token finds its row flagged revoked; and a rotation that lost its conditional revoke is treated the same way. Either way the service revokes every unrevoked refresh token of the row's account (`revokeAllByUser`), emits `AuditAction.REFRESH_TOKEN_REUSE_DETECTED` with `RefreshTokenReuseContext` (the compromised user and the offending token id), and throws `RefreshTokenReuseDetectedException`, a subtype of `InvalidRefreshTokenException`.

**The revoke-all commits in the refresh's own transaction.** `AuthServiceImpl#refreshToken` and `rotateRefreshToken` are both `@Transactional(noRollbackFor = RefreshTokenReuseDetectedException.class)`: the reuse refusal commits, so the revoke-all survives it, and every other refusal rolls back whatever was written before it. Part 2 removed the old `REQUIRES_NEW` transaction, so a refresh holds a single connection and never waits on a second transaction of its own. The reuse audit is published as an event and written by the asynchronous `AFTER_COMPLETION` audit listener in a transaction of its own, so the audit row alone does not prove the revoke-all committed.

**One client message.** Every refusal answers **401 `INVALID_REFRESH_TOKEN`** with the code's default message, "Your session is invalid. Please sign in again.": `InvalidRefreshTokenException#getClientMessage` returns it whichever check failed, and the specific reason stays in `getMessage()`, for the log only. A token holder cannot tell which check refused it.

**Not built yet:** a 30-second grace window for the token a rotation has just replaced, so that a retry after a lost reply is not treated as theft, is decided for the follow-up security PR (the engineering vault, `08 Follow-ups/Security follow-up PR items (pending invite).md`). Until then, any presentation of a rotated token is reuse.

```mermaid
flowchart TD
    A["POST /v1/auth/refresh-token with refresh token"] --> V{"parseRefreshToken accepts it?"}
    V -->|"no"| X["401 INVALID_REFRESH_TOKEN, one message, rolled back"]
    V -->|"yes"| U{"account found and ACTIVE?"}
    U -->|"no"| X
    U -->|"yes"| H["hash token, look up its row"]
    H --> M{"the row"}
    M -->|"none, another account's, or expired"| X
    M -->|"revoked — a replay"| Z["revoke ALL the account's tokens, audit REFRESH_TOKEN_REUSE_DETECTED"]
    M -->|"unrevoked, the owner's, unexpired"| I["insert and flush the new row"]
    I --> C{"conditional revoke of the presented row updates 1?"}
    C -->|"yes"| OK["200 AuthResponse — new access + refresh"]
    C -->|"0, row deleted"| X
    C -->|"0, row still there — lost the race"| D["delete the new row"]
    D --> Z
    Z --> Y["401 INVALID_REFRESH_TOKEN, one message, committed"]
```

On a successful rotate the service also blacklists the old access token for its remaining TTL, refreshes the admin idle key for admin principals, and audits `TOKEN_REFRESH`.

## Sessions & logout

Each refresh token row *is* a session — one per device/login, carrying `deviceInfo`, `ipAddress`, `lastUsedAt`, and its `revoked` flag (`model/user/RefreshToken.java`, table `refresh_tokens`).

| Operation | Endpoint | Effect |
| --- | --- | --- |
| List sessions | `GET /v1/auth/sessions` | Every non-revoked row for the caller as `SessionResponse`; pass `?currentRefreshToken=` to flag the caller's own row `current = true` |
| Revoke one session | `DELETE /v1/auth/sessions/{sessionId}` | Revokes that row; `404 SessionNotFound` if it is not the caller's (never leaks another user's session) |
| Logout (this device) | `POST /v1/auth/logout` | Blacklists the access token; revokes the refresh token supplied in the body only if it is the caller's own (since part 2, #379: another account's token is left as it is, and the answer is still 200); audits `LOGOUT` for the authenticated user |
| Logout everywhere | `POST /v1/auth/logout-all` | `revokeAllUserTokens` + blacklists the current access token; audits with scope marker `all_devices` |

`GET /v1/auth/sessions` maps rows through `mapper/auth/RefreshTokenMapper.java`; the `current` flag is computed by hashing the caller-supplied refresh token and comparing to each stored `tokenHash` (passing no token flags every row not-current). `deviceLabel` is presently just an alias of `deviceInfo`.

The two logout endpoints are distinct routes, not one endpoint with a `scope` parameter — `all_devices` is only an internal audit-scope marker on `LogoutScopeContext`, surfaced through the emitter, not a request field. `logout` tolerates a missing refresh token (it still blacklists the access token); `logout-all` needs no body.

Finally, a **password change** (`POST /v1/auth/change-password`, `AuthServiceImpl#changePassword`) verifies the current password, re-hashes the new one, and calls `revokeAllUserTokens` — every session dies and the user must log in again. Password *reset* (forgot-password flow) does the same. Those flows are detailed in [OTP & password recovery](otp-and-password-recovery.md).

### Cleanup cron

`RefreshTokenServiceImpl#cleanupExpiredTokens` runs every 6 hours (`cron = "0 0 */6 * * *"`) and hard-deletes rows whose `expires_at` has passed — this is what eventually reaps the revoked-but-lingering rows that reuse detection relies on within their window.

## Where it lives

| Concern | Source |
| --- | --- |
| Token minting, claims, HS512 signing, TTL resolution, the one parse per door | `security/jwt/JwtProvider.java` |
| Signing/verification key map + rotation invariants | `security/jwt/JwtKeyProvider.java` |
| Key config binding | `security/config/JwtKeyConfig.java`, `security/config/JwtKeyEntry.java` (`app.jwt.*`) |
| Admin TTL / idle knobs | `security/ratelimit/AdminSessionProperties.java` (`app.security.admin.*`) |
| Per-request auth + admin idle gate | `security/jwt/JwtAuthFilter.java` |
| Authority re-resolution | `security/CustomUserDetailsService.java`, `security/CustomUserDetails.java` |
| Filter ordering, stateless policy, entry point wiring | `security/config/SecurityConfig.java` |
| 401 on missing/invalid token | `security/jwt/JwtAuthEntryPoint.java` |
| Access-token blacklist (Redis) | `security/jwt/JwtBlacklistServiceImpl.java` |
| Refresh rotation, reuse detection, session revocation | `service/auth/RefreshTokenServiceImpl.java` |
| The conditional revoke and the revoke-all | `repository/user/RefreshTokenRepository.java` (`revokeIfNotYetRevoked`, `revokeAllByUser`) |
| The refresh refusals and their one client message | `exception/InvalidRefreshTokenException.java`, `exception/RefreshTokenReuseDetectedException.java` |
| Refresh-token hashing (SHA-256 at rest) | `util/TokenHasher.java` |
| Blacklist-digest hashing (inline SHA-256) | `security/jwt/JwtBlacklistServiceImpl.java` |
| Refresh/logout/session endpoints | `controller/auth/AuthController.java` |
| Session + refresh DTOs | `dto/auth/response/SessionResponse.java`, `dto/auth/request/RefreshTokenRequest.java` |
| Session mapping | `mapper/auth/RefreshTokenMapper.java` |
| Refresh-token entity + table | `model/user/RefreshToken.java` (`refresh_tokens`) |
| Reuse audit context | `service/platform/audit/context/RefreshTokenReuseContext.java` |
| Password bcrypt encoder bean | `security/config/CryptoConfig.java` |

## See also

- [Principal model](principal-model.md) — what the `userId`/`scope`/`ownerType` claims resolve to.
- [Login and the phone gate](login-and-the-phone-gate.md) — the standard login that mints this pair.
- [Admin login and MFA](admin-login-and-mfa.md) — the admin-specific TTLs and the idle-session gate.
- [OAuth](oauth.md) — the one-time-code exchange that ends in the same token pair.
- [RBAC authorization](../rbac/authorization.md) — how per-request authorities are resolved and gated.
- [Audit](../audit/index.md) — where `LOGOUT`, `TOKEN_REFRESH`, and `REFRESH_TOKEN_REUSE_DETECTED` land.
