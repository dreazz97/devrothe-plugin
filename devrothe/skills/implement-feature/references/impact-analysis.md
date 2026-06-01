# Impact analysis (blast radius)

Read in step 3. After the plan is approved and **before changing any code**, tell the user, clearly and
explicitly, how far this feature reaches. Always give a statement — even when the answer is "no impact
on existing features".

## Contents
- What to check
- Security impact
- How to classify
- How to phrase the statement

## What to check

Go through the plan and the mapped code and check whether the feature:

- **Modifies shared code** — utilities, hooks, services, base components, layouts, types reused by
  other features. Changing shared code is the most common way to break things elsewhere.
- **Changes the data model** — migrations, especially destructive ones (drop/rename columns, type
  changes, backfills) or changes to tables other features read/write.
- **Changes API/route contracts** — modifying an existing endpoint's request/response, status codes or
  auth requirements can break current clients.
- **Touches global state/config** — env vars, DI/container, global providers, feature flags, routing.
- **Affects auth/permissions** — new roles/scopes or changes to who can access existing routes.
- **Changes dependencies** — adds/updates/removes packages (version bumps can ripple).
- **Affects build/CI** — new build steps, scripts or pipeline changes.
- **Shares UI surface** — navigation, shared layout, theming that other pages render.

A feature that only adds new files and wires them in through new entry points is low risk; one that
edits shared code or migrations is higher risk.

## Security impact

For a real app (not a PoC/demo — see the gate in `../create-application/references/security.md`),
the impact statement also covers the **new attack surface** the feature introduces. A feature can be
functionally isolated yet add a security risk, so assess it separately. Check whether the feature:

- **Adds an endpoint/route** — does it enforce authentication and an ownership/role check, or is it an
  unguarded entry point or a potential IDOR?
- **Accepts new input** — request bodies, params, query, uploads, webhooks: is each validated at the
  boundary, and are uploads checked for type/size?
- **Makes outbound calls from a user-controlled URL** — SSRF risk; needs an allowlist and blocking of
  internal/metadata hosts.
- **Touches authn/authz** — new login/reset/token flows, roles, or changes to who can reach existing
  routes; needs rate limiting and the auth invariants (`exp`, signature, httpOnly cookies).
- **Handles secrets or PII** — new credentials/keys (env, not code), or personal/financial/health data
  (storage, logging redaction, retention).
- **Adds dependencies** — new packages can carry known vulnerabilities; note them for an audit.

Classify each security finding with a severity (CRITICAL/HIGH/MEDIUM/LOW — see
`../create-application/references/security.md`) so the user can weigh it. If the feature adds no new
attack surface, say so explicitly.

## How to classify

State one of:
- **Isolated / low risk** — only new files; no change to existing behavior. Name what makes it safe.
- **Touches existing features** — list the specific features/areas affected and how.
- **Could break the app** — explain the concrete failure modes (e.g., "the migration renames a column
  three other queries use") and the mitigation.

## How to phrase the statement

Give the user a short, direct statement they can act on:
- the risk level (isolated / touches X / could break Y),
- the **specific** existing features affected (or explicitly "none"),
- for a real app, the **security impact** — new attack surface and its severity, or explicitly "none",
- the mitigations: safety net (branch/commit — see
  `../refactor-application/references/execution.md`), the tests that will guard it (functional and
  security), and whether a feature flag or phased rollout helps.

If the risk is non-trivial — functional **or** security — make sure a reversal path is in place and get
an explicit go-ahead before implementing. Never silently proceed past a "could break the app" or a
HIGH/CRITICAL security finding.
