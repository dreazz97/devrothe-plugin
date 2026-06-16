# Security baseline

Read when building or changing an application that is **not** a throwaway PoC/demo (see the gate
below). This is the shared security source of truth for the devrothe skills: `create-application`
builds it in, `implement-feature` applies it to new code, `refactor-application` aligns to it and
`test-application` covers it. The goal is **secure-by-default construction** — not an audit. Auditing
an existing codebase for vulnerabilities is a different job (a dedicated security-scan skill); here the
point is to *build* so those vulnerabilities never appear.

## Contents
- The PoC/demo gate (when security applies)
- Secure-by-default baseline (cross-cutting)
- OWASP Top 10 — how to build against each
- Secrets and dependency hygiene
- Per-stack setup (Next.js / Express / FastAPI)
- Security testing
- Reporting findings (severities)

## The PoC/demo gate (when security applies)

Security hardening is **conditional**. Decide once, early, whether the project is a real application
or a throwaway, and let that decision drive everything below. Why: hardening a 2-day prototype that
holds no real data and never ships is wasted effort and friction; shipping a real app without it is
negligence. The two cases deserve opposite defaults.

**Skip the hardening (treat as PoC/demo)** when the signals clearly point that way and none of the
production signals are present:
- The user explicitly calls it a PoC, demo, prototype, spike, throwaway, sandbox, hackathon or
  experiment.
- No real users; no real or sensitive data (seed/fake data only).
- Not exposed to the internet / runs locally only; short-lived or disposable.

**Apply the hardening (treat as a real app)** when any production signal is present, *even if the user
called it a "demo"*:
- Real or sensitive data: PII, credentials, financial, health, anything regulated (GDPR/NIS2).
- Authentication, multi-user or multi-tenant; payments; file uploads from users.
- Public/production deployment, real clients, or an explicit "launch"/"production" intent.

**When the signals are ambiguous or conflict, ask the user** (`AskUserQuestion`, single choice):
*"Is this a real application (apply the security baseline) or a throwaway PoC/demo (skip it)?"* Do not
guess on this — the cost of a wrong default is high in both directions.

**Record the decision** so later work honors it without re-asking: note it in the project's `CLAUDE.md`
or `README.md` (e.g., *"Security posture: PoC — hardening intentionally skipped"* or *"…: production —
security baseline applied"*). The other skills read this in their context step (step 0).

**Minimum floor — applies even to PoCs.** Skipping the *hardening* never licenses creating a real-world
liability: never hardcode or commit real secrets/credentials/keys, never paste real production or
personal data into the repo or fixtures, and keep `.env`/`*.pem`/`*.key`/`secrets/` in `.gitignore`.
This is hygiene about the repository, not hardening of the app, so it is not gated.

## Secure-by-default baseline (cross-cutting)

Build these in from the start for a real app — they are cheap up front and expensive to retrofit:

- **Validate every input at the boundary.** Parse-and-validate untrusted input with Zod (TS) /
  Pydantic (Python) at the edge (request bodies, query/params, headers, webhooks, file metadata).
  Reject by default; never trust client-supplied IDs, roles or amounts. Why: validation at the
  boundary is the foundation under injection, broken-access-control and integrity defenses.
- **Authorize, not just authenticate.** Every protected route checks *who* (authenticated) **and**
  *whether they may act on this specific resource* (ownership/role). An authenticated user must not be
  able to read/modify another user's record by changing an `:id`. Why: this is IDOR/A01, the most
  common real breach.
- **Parameterized data access only.** Use the ORM's query API (Prisma / SQLAlchemy); never build SQL
  (or shell commands, or `eval`) by string concatenation/interpolation of input. Why: removes whole
  injection classes structurally.
- **Hash passwords, sign tokens, store sessions safely.** bcrypt/argon2 for passwords (never
  MD5/SHA1); JWTs with `exp` and verified signature (reject `alg:none`); tokens/sessions in `httpOnly`,
  `Secure`, `SameSite` cookies — never `localStorage`. (The `auth` module in `modules.md` already
  encodes this.)
- **Security headers + HTTPS.** Send HSTS (with preload in prod), a real Content-Security-Policy (no
  blanket `unsafe-inline`/`unsafe-eval`), `X-Content-Type-Options: nosniff`, `X-Frame-Options`/frame
  CSP, and a `Referrer-Policy`. Force HTTPS and redirect HTTP→HTTPS in production.
- **Rate-limit the sensitive endpoints.** Login, signup, password reset, token, OTP and upload
  endpoints get rate limiting / lockout. Why: blunts credential stuffing and brute force (A04/A07).
- **Lock down CORS.** Allow the known frontend origin(s) explicitly; never reflect arbitrary origins
  or use `*` with credentials.
- **Fail safe, leak nothing.** Centralized error handling returns generic messages to clients; stack
  traces, SQL and internal details go to logs, never to API responses. Turn debug off in production.
- **Don't log secrets or PII.** Redact tokens, passwords, card data and personal data from logs.
  (The `logging` module already says this — honor it.)
- **Guard outbound requests (SSRF).** If the server fetches a URL derived from user input, validate it
  against an allowlist and block internal/metadata addresses (`169.254.169.254`, `localhost`, private
  ranges). Why: SSRF (A10) turns a fetch into cloud-credential theft.
- **Validate uploads.** Check type, size and (where relevant) content before accepting/serving files;
  store outside the web root or via presigned URLs (see the `storage` module).

## OWASP Top 10 — how to build against each

Use this as the mental checklist while implementing — each row is "what to do", not "what to scan for":

| | Category | Build it this way |
|---|----------|-------------------|
| A01 | Broken Access Control | Auth middleware on protected routes; ownership/role check on every `:id`; deny-by-default. |
| A02 | Cryptographic Failures | bcrypt/argon2 for passwords; TLS on; CSPRNG for tokens (`crypto`/`secrets`), never `Math.random()`. |
| A03 | Injection | ORM/parameterized queries; no `eval`/`pickle`/`yaml.load`(unsafe)/`shell=True`/`os.system` on input. |
| A04 | Insecure Design | Rate limiting on auth/reset/upload; CSRF protection for cookie auth; non-guessable IDs (UUID). |
| A05 | Security Misconfiguration | Locked CORS; real CSP; security headers present; debug off; no default credentials in prod. |
| A06 | Vulnerable Components | Pin & lock deps; audit on install/CI; remove unused deps (see hygiene below). |
| A07 | Auth Failures | JWT `exp` + signature verified; session invalidated on logout; sane password policy; lockout. |
| A08 | Data Integrity | Pinned versions + lockfile committed; verify webhook signatures (e.g. Stripe); no unsafe deserialization. |
| A09 | Logging Failures | Log auth failures and security events; never log PII/secrets; no stack traces in responses. |
| A10 | SSRF | Allowlist outbound URLs from user input; block internal/metadata hosts. |

## Secrets and dependency hygiene

- **Secrets in env, never in code.** Read from environment (validated with Zod / `pydantic-settings`
  at startup — fail fast if missing). Keep `.env`, `*.pem`, `*.key`, `credentials*`, `secrets/` in
  `.gitignore` **before** the first commit. If a secret was ever committed, rotate it — `.gitignore`
  does not remove git history.
- **Runtime/integration credentials may live in the DB — encrypted, never plaintext.** Some real-app
  secrets must be editable at runtime (an admin configures the email provider) or differ per tenant;
  those don't fit a redeploy-to-change `.env`. They go in the **encrypted credential vault** (DB +
  Fernet — AES-128-CBC + HMAC-SHA256; see the `secrets` section in `modules.md`), encrypted at rest and
  decrypted in memory only. This is a deliberate split by lifecycle, not a loophole: bootstrap/infra
  secrets (`DATABASE_URL`, the JWT signing secret, and the **Fernet key** that bootstraps the vault)
  stay in env; only runtime/per-tenant credentials move to the vault. **Plaintext secrets stored in the
  DB are a finding**, the same severity as secrets in code — encryption at rest is the whole point.
- **Dependencies.** Commit the lockfile; pin versions. Run the stack's audit
  (`pnpm audit` / `pip-audit` / `govulncheck` / `cargo audit` / `dotnet list --vulnerable`) and resolve
  known-vulnerable packages: patch bumps apply freely, minor bumps after a changelog check, major bumps
  evaluated for breaking changes. Remove declared-but-unused ("phantom") dependencies.

## Per-stack setup (Next.js / Express / FastAPI)

**Next.js** — set security headers in `next.config` (`headers()`) or `middleware.ts` (CSP, HSTS,
`X-Content-Type-Options`, frame protection, `Referrer-Policy`, `Permissions-Policy`). Validate input in
Server Actions / Route Handlers with Zod. Keep secrets server-side — only `NEXT_PUBLIC_*` reaches the
client; never leak a key through a client component. Rate-limit auth route handlers/actions. Server
Actions carry CSRF protection; for cookie-authenticated Route Handlers add a CSRF token.

**Express** — `helmet` (headers) + `cors` locked to the frontend origin are already in the scaffold;
configure them, don't just install them. Add `express-rate-limit` on auth endpoints, a request body
size limit, Zod validation middleware at the boundary, and a centralized error handler that hides
internals. Data access via Prisma only (no `$queryRawUnsafe` with interpolation).

**FastAPI** — add a security-headers middleware and lock `CORSMiddleware` to known origins (not `["*"]`
with credentials). Pydantic validates input by construction — keep request models strict. Rate-limit
auth routes (e.g. `slowapi`). Data access via SQLAlchemy expressions (no f-string SQL). Don't return
raw exceptions; add an HTTPS-redirect/`TrustedHost` setup for production.

## Security testing

For a real app, security behavior is part of the test suite, not an afterthought — see
`testing.md` for levels and the green gate. Cover, where applicable:
- **Access control**: an authenticated user is **denied** another user's resource (IDOR); an
  unauthenticated request to a protected route is rejected (401/403); role-gated routes reject
  insufficient roles.
- **Input validation**: malformed/oversized/malicious payloads are rejected with a 4xx, not a 500 or a
  silent accept.
- **Auth flows**: login lockout/rate limit triggers; logout invalidates the session; expired/tampered
  tokens are rejected.
- **Webhooks**: an invalid signature is rejected (e.g. Stripe).
Treat the **critical paths** (auth, payments/billing, anything touching PII or tenancy) as must-cover:
missing tests there are high-priority gaps.

## Reporting findings (severities)

When a skill *reports* security issues (e.g. `refactor-application` flagging red flags in existing
code, or stating a feature's security impact), classify each finding so the user can prioritize:
- **CRITICAL** — exploitable now with direct data/security loss (hardcoded prod secret, SQL injection,
  auth bypass, IDOR on sensitive data).
- **HIGH** — exploitable with preconditions, or a real functional security failure (missing authz on a
  sensitive route, tokens in `localStorage`, missing webhook signature check).
- **MEDIUM** — weakens posture but not directly exploitable (missing security headers, permissive CORS,
  weak password policy).
- **LOW** — best-practice/hardening nice-to-haves.
Never invent severities to pad a report; if something can't be assessed, say so.
