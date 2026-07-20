# Security Plan — User Access & Website Security

> Engineering plan for QuietLot (working brand; operated by STY Enterprises, S-corp).
> Covers: identity & access (Amazon Cognito), the invite-only enforcement, application security,
> and platform/data security. Written for the Phase-1 MVP (Next.js + Postgres + Stripe Connect,
> per BLUEPRINT §16) with notes on what applies to the current static prototype.

**Honest status note:** today's site (landing + demo) is a static page with a localStorage demo —
there are no accounts, no server, and nothing sensitive to protect; the demo's "users" are
role-play. Everything below is what we build **when real accounts and real money arrive**, and
nothing sensitive should ever be put into the static demo.

---

## 1. Identity & access — Amazon Cognito

Cognito is a good fit here: managed user pools, MFA, hosted auth flows, Lambda triggers for
custom rules (which is exactly how we enforce invite-only), and SOC/ISO/PCI-compliant
infrastructure without running our own identity stack.

### Architecture

- **One Cognito User Pool** for all humans. A user belongs to an **Account** (business) in our
  Postgres DB; roles (seller/broker/buyer capabilities) live on the Account, not the Cognito
  user — Cognito authenticates *people*, Postgres authorizes *businesses*.
- **Auth flow:** OAuth 2.0 Authorization Code + PKCE via Cognito's hosted UI (or Amplify/
  NextAuth Cognito provider embedded in our design). No implicit flow, no ROPC.
- **Tokens:** short-lived access/ID JWTs (≤1h), rotating refresh tokens, verified server-side
  (issuer + audience + signature via JWKS) on every API call. Tokens stored in httpOnly,
  Secure, SameSite=Lax cookies — never localStorage.
- **MFA:** TOTP required for account owners and any user with payout/bank permissions;
  optional-but-nudged for everyone else. SMS as fallback only.
- **Invite-only enforcement (the important custom part):** a **Pre-Sign-Up Lambda trigger**
  rejects any registration without a valid, unexpired, unused invite code (from a seller/broker
  invite or an approved access request). No code, no account — signup is closed at the identity
  layer, not just hidden in the UI.
- **Adaptive protection:** Cognito advanced security ON — risk-based blocking of credential-
  stuffing, compromised-credential checks, unusual-location step-up MFA.
- **Account recovery:** email-based with mandatory re-verification; payout-capable users
  additionally re-verify via MFA before bank/payout changes take effect (plus a 24h hold +
  notification on payout-destination changes — classic fraud pattern).
- **Org membership:** multiple users per Account with owner/manager/viewer roles; owner-only:
  relationship terms, identity reveal, payout settings, user management.

### Authorization (separate from authentication)

- Every domain query is scoped by the **relationship graph** — enforced in **Postgres
  row-level security** so "what can this account see" is answered once, at the data layer.
  This is the control that makes identity-leak bugs structurally hard (BLUEPRINT §16).
- API layer re-checks entitlements per mutation (defense in depth over RLS).
- The masking rule is a **server-side view concern**: raw legal identity never leaves the API
  unless the viewer holds a reveal grant — aliases are substituted server-side, so no client
  ever possesses data it shouldn't render.

## 2. Application security (the website itself)

- **Headers/CSP:** strict Content-Security-Policy (self-only scripts, no inline eval),
  HSTS, X-Frame-Options DENY, Referrer-Policy strict-origin, Permissions-Policy minimal.
- **Input handling:** parameterized queries only (no string SQL); output encoding everywhere;
  file uploads (listing images, manifests) type-sniffed, size-capped, virus-scanned, stored in
  S3 with private ACLs and served via signed URLs on our domain — never raw user URLs.
- **CSRF:** SameSite cookies + double-submit token on state-changing routes.
- **Rate limiting & abuse:** per-IP and per-account limits on auth, messaging, access requests,
  and search (enumeration of teasers/aliases is an attack on the masking model — treat
  scraping as a security event, not a nuisance). WAF (Vercel/Cloudflare) in front.
- **Secrets:** never in the repo; environment secrets in Vercel/AWS Secrets Manager, rotated;
  least-privilege IAM for the Lambda triggers and S3.
- **Dependencies:** lockfiles + Dependabot/audit in CI; no third-party scripts on
  authenticated pages (also an identity-leak vector via referrer/pixel).
- **Payments:** all card/bank data lives at Stripe (SAQ-A posture) — payment details never
  touch our servers.

## 3. Data & platform security

- **Encryption:** TLS everywhere; encryption at rest (RDS/S3 defaults + KMS); field-level
  encryption for the few high-sensitivity columns (legal identity ↔ alias mapping table).
- **The crown jewel is the alias↔identity mapping and the relationship graph.** Access to it in
  production is service-account-only, human access break-glass with logging.
- **Audit log:** append-only events for relationship changes, reveals, terms edits, routing
  decisions, payout changes, admin actions — this is both a security control and the dispute
  evidence base (MARKET-TERMS §6).
- **Backups & DR:** automated Postgres PITR backups, restore drill quarterly.
- **Monitoring:** structured logs, alerting on auth anomalies (Cognito events → CloudWatch),
  error tracking, uptime checks. Security incident runbook: contain → assess scope via audit
  log → notify affected accounts honestly → post-mortem.
- **Privacy posture:** collect the minimum; the demo collects nothing (localStorage only);
  formal privacy policy replaces the draft before real accounts exist.

## 4. Sequencing

| When | What ships |
|---|---|
| **Now (static site)** | Draft ToS/Privacy pages live; no real auth (nothing to protect); headers via Vercel config |
| **MVP week 1** | Cognito pool + hosted UI + pre-signup invite Lambda; httpOnly sessions; RLS schema from day one |
| **MVP week 2–3** | MFA enforcement for payout roles; rate limits; WAF; audit log; S3 signed-URL uploads |
| **Before first real order** | Pen-test pass (even a lightweight one), incident runbook, backup restore drill |
