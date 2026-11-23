---
name: cybersec-checklist
description: Detailed security checklist + post-audit gates for any change touching auth, payments, encryption, user input, secrets, dependencies, public endpoints, or PII flows. Load BEFORE writing security-sensitive code or BEFORE merging a PR that modifies auth/data-handling.
---

# CYBERSEC checklist — full procedure

The procedure to follow for any change that touches auth, payments, encryption, user input, secrets, dependencies, public endpoints or personal data. Your project's own rules (for example in `CLAUDE.md`) come first; this checklist fills the gaps.

## Pre-audit (cybersec layer)

1. **Threat-model the change** — what's user-controlled vs server-controlled? what data class flows (PII / credential / business / public)? OWASP top-10 sweep on touched code.
2. **Map existing controls** — auth middleware, input validation, rate limit, audit trail, encryption. Verify they're still intact post-fix.

## Code-level requirements (every fix)

1. **Validate every inbound payload** — Zod / Joi on body, query, headers, file uploads. Server validates, never trusts client.
2. **Parameterize every query** — no string concat / template literals with user input in SQL. No `$queryRawUnsafe(...userInput...)`.
3. **Auth + authz on every protected route** — fail closed. Verify the user owns / can act on the resource (no ID-swap via URL or token-claim drift).
4. **Use platform crypto only** — `node:crypto` / WebCrypto `subtle`. `randomUUID` / `randomBytes` for secrets. `timingSafeEqual` for comparisons. No homemade crypto.
5. **Webhooks need 5-layer defense** — network ACL → schema validation → crypto signature (HMAC-SHA256/512, RFC-9421) → time-window check (replay protection) → idempotency key. All five are mandatory; skipping any one collapses the security model.
6. **Rate-limit every public endpoint** — per-IP and per-user where applicable.
7. **TLS everywhere** — no plain HTTP between services or to third parties.
8. **Errors fail closed** — auth / validation error → 4xx with generic message + correlation ID. Stack traces stay server-side.
9. **No client-side enforcement only** — every UI gate has a server check.

## Secrets discipline

1. **Never paste secrets** in chat, terminal, commits, logs. Read from `process.env` only.
2. **Never commit `.env*`** — only `.env.example` with placeholders. Verify `.gitignore` before staging.
3. **If a secret leaks** (transcript, terminal echo, repo, screenshot) — STOP. Rotate immediately. Document rotation in the changelog. No exceptions.

## Logging + audit trail

1. **Log security events** with correlation ID — auth failures, rate-limit hits, signature mismatches, permission denials, suspicious payloads.
2. **Redact PII / secrets** in logs — phone, email, password, token, address, full DB URLs. Hash if correlation is needed.
3. **Audit trail for sensitive ops** — payments, password / role changes, data exports → write to a separate immutable log (Sentry breadcrumbs, dedicated audit table, or both).

## Dependencies (supply chain)

1. **Scan before adding** — `sonatype-guide` / `aikido` / Endor Labs (`ai-plugins`), all installed by this kit. Check vuln history, maintainer reputation, last release date.
2. **Pin exact versions** for security-critical packages (crypto, auth, validation, signing). No `^` for these.
3. **`npm audit --omit=dev` clean** before merging to main.

## Post-audit cybersec checklist

Run through this list before every PR merge :

- [ ] No secret in the diff (sources, lockfile, env-example, commit message)
- [ ] No `console.log` of `req.body` or sensitive vars
- [ ] No raw SQL with concat / template-literal of user input
- [ ] Every new public route has auth + rate-limit + input validation
- [ ] No fail-open paths (errors return 4xx, not 200)
- [ ] No new dependency added without scan
- [ ] No CSP / CORS / security-header regression
- [ ] No `--no-verify` / signing skips in commits
- [ ] No reduction in encryption / hashing / iteration counts vs prior code
