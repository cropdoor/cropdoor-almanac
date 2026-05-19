# Security

How CropDoor protects authentication, sessions, and request integrity.

The security stack is built around stateless JWTs (HS512), Spring Security's filter chain, Redis-backed rate limiting, per-request correlation IDs, and a strict CORS allow-list. Everything below the auth filter assumes the caller's identity has already been resolved into a `CustomUserDetails`.

## Planned pages

- **Security overview** — the Spring Security filter chain, what each filter does, the order they run in (`SecurityConfig.java`).
- **JWT and key rotation** — `JwtProvider`, `JwtKeyProvider`, `JwtKeyEntry`, `JwtBlacklistService`. How access + refresh tokens are minted, validated, and revoked.
- **OAuth (Google)** — the social-login flow: `RoleAwareOAuth2AuthorizationRequestResolver`, `OAuthRoleStateStore`, success/failure handlers, the `OAUTH_SIGNUP` vs `OAUTH_LINKED` discrimination.
- **Rate limiting** — `RedisRateLimitService`, `ScopedRateLimiter`, `RateLimitFilter`, the per-admin per-identifier filter `AdminPerUserRateLimitFilter`.
- **Password policy and lockout** — failed-attempt threshold, lockout window, the `ACCOUNT_LOCKED` audit path.
- **Correlation ID** — `CorrelationIdFilter`, the `X-Request-Id` header, how it flows into the MDC and logs.
- **CORS** — `CorsConfig`, `CorsProperties`, the `CORS_ALLOWED_ORIGINS` env var.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/security/`.
