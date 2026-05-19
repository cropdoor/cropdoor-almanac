# Notifications

How CropDoor sends email, SMS, and voice OTP — and how delivery status flows back from third-party providers.

Three transports, each with a pluggable sender bean (`Noop` for local dev, real provider for prod). OTP dispatch branches on `OtpChannel`. Email goes through `Resend` with templated Thymeleaf bodies. SMS and delivery callbacks go through `Arkesel`.

## Planned pages

- **Email pipeline** — `ResendConfig`, `ResendProperties`, the email-event listener, the architectural pin against direct send calls.
- **SMS pipeline** — the Arkesel sender, the `SMS_PROVIDER=noop` vs `arkesel` switch, the `ARKESEL_CALLBACK_ENABLED` flag.
- **Voice OTP pipeline** — `NoopVoiceSender` (default in non-prod), `ArkeselVoiceSender` (flagged as broken scaffolding pending rewrite to `/api/otp/generate`).
- **OTP routing** — `sendOtp` channel branching, the typed verify outcome (`AUTHENTICATED` vs `PHONE_VERIFIED`), `OTP_SENT` / `OTP_VERIFIED` / `OTP_FAILED` audits.
- **Delivery webhooks** — the Arkesel callback at `controller/webhook/`, the `ARKESEL_WEBHOOK_PATH_SECRET` URL-safe secret, how `otp_codes.sms_delivery_status` updates.

!!! info "Status"
    Scaffolding. Source material under `src/main/java/com/cropdoor/backend/service/notification/` (`email/`, `sms/`, `voice/`), `event/email/`, `controller/webhook/`.
