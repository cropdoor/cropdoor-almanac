# Security

How CropDoor protects authentication, sessions, and request integrity.

The security stack is built around stateless JWTs (HS512), Spring Security's filter chain, Redis-backed rate limiting, per-request correlation IDs, and a strict CORS allow-list. Everything below the auth filter assumes the caller's identity has already been resolved into a `CustomUserDetails`. There is no session store; `Authorization: Bearer <token>` is the only auth channel.

## Planned pages

- **Security overview** — the Spring Security filter chain, what each filter does, the order they run in (`SecurityConfig.java`).
- **JWT and key rotation** — `JwtProvider`, `JwtKeyProvider`, `JwtKeyEntry`, `JwtBlacklistService`. How access + refresh tokens are minted, validated, and revoked.
- **OAuth (Google)** — the social-login flow: `RoleAwareOAuth2AuthorizationRequestResolver`, `OAuthRoleStateStore`, success/failure handlers, the `OAUTH_SIGNUP` vs `OAUTH_LINKED` discrimination.
- **Rate limiting** — `RedisRateLimitService`, `ScopedRateLimiter`, `RateLimitFilter`, the per-admin per-identifier filter `AdminPerUserRateLimitFilter`.
- **Password policy and lockout** — failed-attempt threshold, lockout window, the `ACCOUNT_LOCKED` audit path.
- **Correlation ID** — `CorrelationIdFilter`, the `X-Request-Id` header, how it flows into the MDC and logs.
- **CORS** — `CorsConfig`, `CorsProperties`, the `CORS_ALLOWED_ORIGINS` env var.

## Credentials never print

Since #397 (19 Nov 2026), no record or class that holds a credential prints it. A credential is a password, an OTP or handoff code, an access, refresh, claim or verification token, an API key or secret, a presigned link, a stored password or code hash, or a provider's raw payload, whatever the field is called.

Why it matters: a Java record's generated `toString` prints every component. With `org.springframework.web` at DEBUG or TRACE, the usual first step when debugging a request, Spring logs each request body it reads through that `toString`, so a single log-level switch would write passwords and codes into the app log. The same change also stopped five log paths that wrote a credential at the default levels.

How the rule is kept:

- **Records.** A record's `toString` is one line through the shared helper `util/Redaction.java`, for example `return Redaction.redactedText(this, "password");`. It names every credential component, and each one prints as `<redacted>`, set or null. The other components print as the generated text would, so a log line still shows which fields came in.
- **Lombok classes.** A `@Data`, `@Value` or `@ToString` class marks a credential field `@ToString.Exclude`.
- **Builders.** A `@Builder` on such a record or class declares its `<Name>Builder` class, whose `toString` prints `<redacted>`. The builder Lombok generates prints every field and ignores `@ToString.Exclude`; `Order`, `Notification`, `User`, `AuthResponse` and others declare theirs.
- **Logs and exceptions.** A log line, an exception message and an exception's cause never name a credential.
- **Older masks stay.** Texts that already mask in their own words keep their shapes: `HandoffCode.masked`, `OrderResponse.DeliveryPhoto`, `RenderedMessage` and `RefreshTokenService.IssuedSession`.
- **Not a credential.** A value that opens nothing on its own: a SHA-256 hash of a high-entropy token (`RefreshToken.tokenHash`), an S3 key, a lease fence.

The rule is review-enforced: it is written in the backend's `CLAUDE.md` ("Credentials never print"). A CI check is planned, not built.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/security/`. Sub-pages will get split out as each topic is documented in depth.
