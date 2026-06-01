---
name: implement-feature
description: Implements a user-requested feature while respecting the project's existing conventions and the other devrothe skills' best practices. First asks clarifying questions if the request is ambiguous; then always produces a plan (with an extensive codebase analysis when the conversation lacks enough context) for approval; before coding, states the feature's impact — whether it can break the app, which existing features it touches, or that it is isolated; then implements it slice by slice with real tests, preserving existing behavior. Use when the user wants to add or build a specific feature in an existing project. Triggers on "implement-feature", "implement a feature", "add a feature", "build this feature", and in Portuguese "implementa esta feature", "implementa uma funcionalidade", "adiciona uma funcionalidade", "nova feature".
---

# implement-feature

Implement a feature the user asked for, into an existing project, respecting its conventions and the
methodology of the other devrothe skills (`create-application` for stack/structure/tests,
`test-application` for the test flow). Understand first, plan, disclose impact, then build with real
tests.

**Rule: follow the project's conventions and the target methodology.** Place code in the existing
feature-first structure, reuse the project's stack and patterns, and align with
`../create-application/references/web-stack.md` / `app-stack.md` (structure) and `modules.md`
(cross-cutting). Why: a feature that ignores the project's conventions becomes tech debt the moment it
lands.

**Rule: preserve existing behavior.** The new feature must not change or break other features. Assess
the blast radius (step 3) and keep changes isolated; touch shared code only when necessary and with a
safety net. Why: the user wants an addition, not regressions elsewhere.

**Rule: real tests + manual-testing disclaimer.** Ship real tests for the feature (see
`../create-application/references/testing.md`) and, when reporting, tell the user which checks are
still recommended to do manually — especially UI/UX in a real browser. Why: a green suite confirms
logic, not look and feel.

**Rule: professional UI/UX, not the generic AI look.** If the feature has any UI, follow
`../create-application/references/design.md` — professional UI/UX fundamentals (hierarchy, spacing,
type scale, WCAG AA contrast, focus/keyboard states, responsiveness) and none of the AI-slop tells
(emojis as UI, cliché purple→blue gradients, untouched stock defaults, buzzword filler). Match the
project's existing design language. Why: a new feature must look as deliberate and consistent as the
rest of the app, not mass-produced.

**Rule: secure-by-default — unless the project is a PoC/demo.** Build the feature to the security
baseline of the project it lands in (`../create-application/references/security.md`): validate new
input at the boundary, authorize/ownership-check any new protected route, use parameterized data
access, guard new outbound calls (SSRF) and uploads, and keep secrets out of code. This is **gated** on
the project's security posture (read it from CLAUDE.md/README in step 0; if it is a real app the
baseline applies, if it is a PoC/demo the hardening is skipped beyond the repo-hygiene floor; ask the
user when it is ambiguous). The feature's **security impact** is part of the impact statement (step 3).
Why: a new feature is the most common place fresh vulnerabilities enter a previously-clean app, and it
should not lower the bar the surrounding code already meets.

## Workflow

Copy this checklist into the response and tick items off:

```
- [ ] 0. Context — read CLAUDE.md, memory, READMEs/docs; understand the codebase area, conventions and security posture (PoC/demo or real app)
- [ ] 1. Clarify — ask the user questions if the request is ambiguous (skip if already clear)
- [ ] 2. Plan — analysis (extensive if the conversation lacks context) + a plan; present for approval
- [ ] 3. Impact — after approval, before coding: state the blast radius (can it break the app? which features it touches, or that it is isolated) + the security impact
- [ ] 4. Implement — slice by slice, following the project's stack/structure/conventions (secure-by-default for a real app), with a safety net for changes to existing code
- [ ] 5. Tests & validate — real tests for the feature (incl. security-relevant ones), green suite + lint/build/startup, + manual-testing disclaimer
```

### 0. Project context

Before anything, look for and read the metadata that explains the project and its conventions:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** (preferences and decisions already recorded).
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs and notes.

Also map the part of the codebase the feature will live in (stack, folder structure, existing patterns)
so the new code blends in. Why: the feature must match what is already there, not impose a new style.

Determine the project's **security posture** here too: read it from CLAUDE.md/README if recorded;
otherwise infer it (does the app handle real users/data/payments or is it a throwaway PoC/demo?) the
way `../create-application/references/security.md` describes, and **ask the user if it is ambiguous**.
This decides whether the secure-by-default rule and the security-impact statement apply.

### 1. Clarify

If the feature request is ambiguous or under-specified, ask the user clarifying questions with
`AskUserQuestion` before planning — scope, expected behavior, inputs/outputs, edge cases, UI
expectations, data involved, who can use it. If the conversation already makes the feature clear, skip
this step. Do not guess on decisions that change the design.

### 2. Plan

**Always produce a plan.** If the conversation does not already contain enough context about the
feature and the affected code, first run an **extensive codebase analysis** (see
`references/analysis-and-plan.md`): where the feature fits, the data model/migrations, routes/endpoints,
shared modules/components it will use or change, and integration points (auth, storage, external
services).

Turn it into a plan of **small vertical slices** ordered by dependency, noting for each what it adds
and which tests cover it. Record the slices as tasks with `TaskCreate`. Present the plan and ask for
approval **before** coding.

### 3. Impact statement (mandatory, after approval, before coding)

Once the plan is approved, and **before changing any code**, give the user a clear, explicit impact
statement (see `references/impact-analysis.md`):
- whether the feature **could break the app** and how,
- **which existing features it touches** (list them), or
- that it is **isolated** and does not affect any existing feature,
- and, for a real app, the feature's **security impact** — new attack surface it introduces (a new
  endpoint, input, external call, upload, dependency, or handling of secrets/PII) and how it is
  covered, or explicitly that it adds none.

Always state this even when the answer is "no impact". If the risk is non-trivial (functional or
security), ensure a safety net (branch/commit) and get an explicit go-ahead before proceeding.

### 4. Implement

Implement the plan slice by slice, placing code in the project's feature-first structure and following
its conventions. For changes that touch existing code, work with a **safety net** (a git branch and
commits per slice, or a backup if the project has no git — see
`../refactor-application/references/execution.md`). Keep other features' behavior unchanged. For any UI,
follow `../create-application/references/design.md` (professional UI/UX, no AI-slop tells) and match the
project's existing design language. For a real app, apply the secure-by-default baseline to the new
code (`../create-application/references/security.md`): boundary validation, authorization/ownership
checks, parameterized data access, SSRF/upload guards — matching the project's existing security
patterns rather than inventing parallel ones. Update the `README.md` and the tasks (`TaskUpdate`) as the
feature lands.

### 5. Tests & validate

Follow the `test-application` methodology for the feature's tests:
- Detect the project's test setup (see `../test-application/references/detect-and-run.md`); reuse the
  existing framework or install the idiomatic one if missing.
- Write **real tests** for the feature at the right level (unit/integration/component/e2e — see
  `../create-application/references/testing.md`): happy path + relevant errors/edge cases. For a real
  app, include the security-relevant cases for what the feature added (see "Security testing" in
  `../create-application/references/security.md`): e.g. a denied cross-user/unauthenticated access on a
  new protected route, a rejected malformed/oversized payload.
- Run the affected tests and the **full suite**; it must end green, plus lint, build and startup. Fix
  before calling it done.
- **Manual-testing disclaimer**: tell the user which checks are still recommended to do manually,
  especially UI/UX interaction in a real browser, since graphical aspects are hard to verify reliably
  with AI.

Finish with a short report: what was added, which files were created/changed, the impact on existing
features, the test results and the recommended manual checks.

## References

- **`references/analysis-and-plan.md`** — how to analyze the codebase to fit the feature and structure
  the plan. Read in step 2.
- **`references/impact-analysis.md`** — how to assess the blast radius and phrase the impact statement.
  Read in step 3.
- **Target methodology** (reused from other skills): `../create-application/references/web-stack.md`,
  `app-stack.md`, `modules.md`, `testing.md`, `design.md` (UI/UX),
  `security.md` (secure-by-default baseline + the PoC/demo gate, applied in steps 0, 3, 4 and 5);
  `../test-application/references/detect-and-run.md`; `../refactor-application/references/execution.md`
  (safety net).
