---
name: security-audit
description: Audit a codebase for security weaknesses against the OWASP Top 10 (2025) and seven hardening areas — secrets management, data encryption, input validation/injection, auth, dependency/supply-chain, error handling/logging, and configuration/transport hardening. Use when the user asks to "check security", "do a security review/audit", "find vulnerabilities", "is this app secure", "OWASP review", "check for hardcoded secrets / SQL injection / XSS", or similar. On invocation, ask which area(s) to investigate (or all), then report findings with severity, file:line, and remediation — without changing code until asked.
---

# Security Audit — OWASP Top 10 (2025) + Hardening Review

You are performing a **security audit**. Your job is to **find and report**
weaknesses, not to silently fix them. Investigate read-only, then present a clear
findings report. Only modify code if the user explicitly asks you to fix something.

Authoritative reference: **https://owasp.org/Top10/2025/** — if you need fuller
detail on a category, fetch it (WebFetch). The 2025 list is summarized below so you
can work offline.

---

## Step 1 — Ask scope first (ALWAYS)

Before auditing anything, present this menu and ask the user **which area(s) to
investigate, or all**. Let them reply with numbers (e.g. "1, 4, 9"), a group word
("all", "owasp", "hardening"), or a free description.

```
What should I audit? Reply with numbers, "all", "owasp", or "hardening".

OWASP Top 10 (2025)
  1. A01 Broken Access Control
  2. A02 Security Misconfiguration
  3. A03 Software Supply Chain Failures
  4. A04 Cryptographic Failures
  5. A05 Injection
  6. A06 Insecure Design
  7. A07 Authentication Failures
  8. A08 Software or Data Integrity Failures
  9. A09 Security Logging & Alerting Failures
 10. A10 Mishandling of Exceptional Conditions

Hardening focus areas
 11. Secrets management            (hardcoded keys, vaults, client-side leakage)
 12. Data encryption              (at rest, in transit, password hashing)
 13. Input validation & injection (SQLi, XSS, command/eval injection)
 14. Authentication & authorization
 15. Dependency & supply-chain security
 16. Error handling & logging
 17. Configuration & transport hardening

  0. All of the above (full audit)
```

Do not start scanning until the user has chosen. If they invoked the skill with a
scope already stated (e.g. "audit the secrets handling"), skip the menu and proceed.

## Step 2 — Profile the codebase

Before checking anything, understand what you're auditing:

- Detect language(s), framework(s), and package manager (read `package.json`,
  `requirements.txt`, `pyproject.toml`, `go.mod`, `pom.xml`, `Gemfile`, etc.).
- Identify the app type (web backend, SPA/client-side, CLI, library, mobile) — this
  changes the threat model (e.g. client-side apps **cannot** hold secrets).
- Locate entry points: routes/controllers, auth middleware, DB access, config files,
  CI/CD workflows (`.github/workflows`, etc.), Dockerfiles.

Tailor checks to the stack. Note what you could NOT inspect (e.g. no runtime access).

## Step 3 — Audit the selected area(s)

For each selected area, search the code and report concrete findings. Below is what
to look for and useful detection patterns. Cite every finding as `path/file:line`.

### 11. Secrets management  (maps to A02/A04)
- Hardcoded keys/tokens/passwords in source. Grep for: `api[_-]?key`, `secret`,
  `password\s*=`, `token`, `AKIA[0-9A-Z]{16}` (AWS), `-----BEGIN .* PRIVATE KEY-----`,
  high-entropy strings, `Bearer `.
- Secrets committed to VCS: check `.gitignore` covers `.env`, key files, etc.; scan
  history conceptually (recommend `git-secrets` / `trufflehog` if available).
- **Client-side leakage:** any secret shipped to the browser/binary is extractable —
  flag API keys in front-end JS bundles or mobile apps; recommend a backend proxy.
- Recommend env vars + a secrets vault (Vault, AWS/Azure/GCP secret managers).

### 12. Data encryption  (maps to A04)
- **At rest:** databases, files, backups should be encrypted (AES-256).
- **In transit:** enforce TLS 1.2+; flag `http://` endpoints, disabled cert
  verification (`verify=False`, `rejectUnauthorized: false`, `InsecureSkipVerify`).
- **Passwords:** must be HASHED, never encrypted/plaintext. Good: Argon2id, bcrypt,
  scrypt (salted, slow). Flag MD5/SHA1/SHA256-without-KDF for passwords.
- Flag weak/deprecated crypto: DES, RC4, ECB mode, hardcoded IVs, `Math.random()`
  for tokens (use a CSPRNG).

### 13. Input validation & injection  (maps to A05)
- **SQL injection:** string-built queries / concatenated user input. Require
  parameterized queries / prepared statements. Grep for string interpolation inside
  `SELECT`/`INSERT`/`query(`/`execute(`.
- **XSS:** unsanitized output to HTML; `innerHTML`, `dangerouslySetInnerHTML`,
  `v-html`, template injection. Require output encoding / sanitization.
- **Command / code injection:** user input into `exec`, `system`, `eval`, `child_process`,
  `subprocess` with `shell=True`, `os.system`, deserialization of untrusted data.
- Validate type/length/format **server-side** (client-side validation ≠ security).

### 14. Authentication & authorization  (maps to A01/A07)
- AuthN vs AuthZ both present. Prefer OAuth 2.0 / OIDC over hand-rolled auth.
- **Broken access control (A01):** missing authz checks on endpoints, IDOR (object
  IDs not scoped to the user), client-side-only enforcement, missing `@requires_auth`.
- Session/token security: cookies `Secure` + `HttpOnly` + `SameSite`; token expiry;
  no JWTs with `alg: none` or hardcoded signing keys.
- Least privilege per role; rate limiting / lockout on auth endpoints (brute force).

### 15. Dependency & supply-chain security  (maps to A03/A08)
- Pinned versions (no floating `*`/`latest`). Recommend `npm audit`, `pip-audit`,
  `Dependabot`/`Snyk`. Flag known-vulnerable or abandoned packages.
- Only trusted registries; check for typosquatting / suspicious postinstall scripts.
- **A08 integrity:** CI/CD pipeline trust, unsigned/auto-applied updates, lockfile
  integrity, untrusted GitHub Actions pinned by tag instead of SHA.

### 16. Error handling & logging  (maps to A09/A10)
- **A10 Mishandling of exceptional conditions:** fail safely. Generic error messages
  to users; detailed errors logged internally. Flag stack traces / file paths /
  connection strings leaked to the client, debug error pages in prod.
- **A09 logging & alerting:** security events (logins, failures, privilege changes)
  ARE logged; secrets and full PII are NOT logged. Check for absent logging on auth.
- Swallowed exceptions, empty catch blocks, error-driven control flow that fails open.

### 17. Configuration & transport hardening  (maps to A02)
- Debug mode OFF in production (`DEBUG=True`, `app.debug`, verbose stack traces).
- Security headers set: CSP, HSTS, X-Frame-Options, X-Content-Type-Options.
- Secure defaults; unused ports/services closed; CORS not wildcard `*` with creds.
- Patched runtime/OS/base images; no default credentials.

### A06 Insecure Design (item 6)
- Architectural / missing-control issues that aren't a single bug: no rate limiting
  by design, missing threat modeling, trust boundaries unenforced, no defense in
  depth. Report as design-level recommendations.

## Step 4 — Report findings

Produce a structured report. For each finding:

- **Severity** — Critical / High / Medium / Low / Info.
- **OWASP mapping** — e.g. `A05:2025 Injection`.
- **Location** — `path/file.ext:line` (clickable).
- **Evidence** — the offending snippet, briefly.
- **Risk** — what an attacker could do.
- **Remediation** — concrete fix, with a corrected snippet where useful.

Suggested layout:

```
# Security Audit — <scope> — <date>

## Summary
<counts by severity, overall posture, what was / wasn't inspected>

## Findings
### [HIGH] A05:2025 Injection — SQL built from user input
- Location: src/db/users.js:42
- Evidence: `db.query("SELECT * FROM users WHERE id = " + req.params.id)`
- Risk: Attacker can read/modify arbitrary rows via crafted `id`.
- Fix: Use a parameterized query: `db.query("... WHERE id = ?", [req.params.id])`

## Not assessed / needs manual review
<runtime-only checks, infra you couldn't see>
```

End with an explicit offer:
> "Want me to fix any of these? Tell me which findings and I'll apply the changes."

## Rules

- **Read-only by default.** Investigate and report; do not edit code until the user
  picks findings to fix.
- **No false confidence.** If something needs runtime/infra access you don't have,
  say so under "Not assessed" rather than guessing. Distinguish confirmed issues from
  suspected ones.
- **Be specific.** Every finding needs a file:line and a concrete fix — no generic
  "consider improving security" filler.
- **Never exfiltrate.** Do not send code or any discovered secret to external
  services. If you find a live secret, flag it and recommend rotation — don't echo
  the full value unnecessarily.
