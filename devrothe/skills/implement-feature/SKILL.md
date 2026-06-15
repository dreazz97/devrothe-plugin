---
name: implement-feature
description: Implements one or more user-requested features while respecting the project's existing conventions and the other devrothe skills' best practices. Handles a single feature or a list of them: clarifies any ambiguous ones; then always produces a plan (with an extensive codebase analysis when the conversation lacks enough context) covering every feature, ordered by dependency, for approval; before coding, states each feature's impact — whether it can break the app, which existing features it touches, or that it is isolated; then implements them one at a time, slice by slice with real tests, preserving existing behavior. Use when the user wants to add or build one or several features in an existing project. Triggers on "implement-feature", "implement a feature", "implement these features", "add a feature", "build this feature", and in Portuguese "implementa esta feature", "implementa estas features", "implementa uma funcionalidade", "adiciona uma funcionalidade", "nova feature", "lista de funcionalidades".
---

# implement-feature

Implement one or more features the user asked for, into an existing project, respecting its conventions
and the methodology of the other devrothe skills (`create-application` for stack/structure/tests,
`test-application` for the test flow). The request may be a single feature or a list of them — treat it
as a **feature set**. Understand first, plan, disclose impact, then build with real tests.

**Rule: one feature or a list — plan the whole set, ship them one at a time.** When the user asks for
several features, do not fan out and build them in parallel: capture them all, plan them together
ordered by dependency (a feature others build on comes first), then implement and validate them **one
feature at a time** — each as its own set of vertical slices with its own green tests before starting the
next. Why: a single consolidated plan lets the user see and approve the whole scope and the right order,
while sequential delivery keeps each feature isolated, individually reviewable and revertible instead of
a tangled, half-done batch. A request for a single feature is just a set of one — the same flow applies.

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
type scale, WCAG AA contrast in both light and dark, focus/keyboard states, responsiveness) and none of the AI-slop tells
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
- [ ] 1. Clarify — list out the requested feature(s); ask the user questions about any that are ambiguous (skip if already clear)
- [ ] 2. Plan — analysis (extensive if the conversation lacks context) + one plan covering every feature, ordered by dependency; present for approval
- [ ] 3. Impact — after approval, before coding: state the blast radius per feature (can it break the app? which features it touches, or that it is isolated), the cross-feature interactions + the security impact
- [ ] 4. Implement — feature by feature in dependency order, each one slice by slice following the project's stack/structure/conventions (secure-by-default for a real app), with a safety net; checkpoint between features
- [ ] 5. Tests & validate — real tests per feature (incl. security-relevant ones), green suite + lint/build/startup after each feature and at the end, + manual-testing disclaimer
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

First, restate the **feature(s)** you understood from the request as an explicit list, so the user can
confirm the scope — this matters most when the request bundles several features or is a vague "build X,
Y and Z". Then, for any feature that is ambiguous or under-specified, ask clarifying questions with
`AskUserQuestion` before planning — scope, expected behavior, inputs/outputs, edge cases, UI
expectations, data involved, who can use it. If the request is a long list, also confirm priority and
whether any items are out of scope for now. If the conversation already makes every feature clear, skip
this step. Do not guess on decisions that change the design.

### 2. Plan

**Always produce a plan** — one consolidated plan that covers every requested feature. If the
conversation does not already contain enough context about the feature(s) and the affected code, first
run an **extensive codebase analysis** (see `references/analysis-and-plan.md`): where each feature fits,
the data model/migrations, routes/endpoints, shared modules/components it will use or change, and
integration points (auth, storage, external services).

Turn it into a plan of **small vertical slices**. When there are several features, **order the features
themselves by dependency** (a feature others build on comes first; flag features that overlap or touch
the same code) and group the slices under each feature, so the plan reads as an ordered backlog rather
than one undifferentiated list. Note for each slice what it adds and which tests cover it. Record the
slices as tasks with `TaskCreate` (one task group per feature). Present the plan and ask for approval
**before** coding.

### 3. Impact statement (mandatory, after approval, before coding)

Once the plan is approved, and **before changing any code**, give the user a clear, explicit impact
statement (see `references/impact-analysis.md`). State it **per feature** (a one-liner each when there
are several):
- whether the feature **could break the app** and how,
- **which existing features it touches** (list them), or
- that it is **isolated** and does not affect any existing feature,
- and, for a real app, the feature's **security impact** — new attack surface it introduces (a new
  endpoint, input, external call, upload, dependency, or handling of secrets/PII) and how it is
  covered, or explicitly that it adds none.

When the set has more than one feature, also call out **cross-feature interactions** — features in the
batch that depend on each other or modify the same shared code/data model — since those compound the
blast radius. Always state this even when the answer is "no impact". If any feature's risk is non-trivial
(functional or security), ensure a safety net (branch/commit) and get an explicit go-ahead before
proceeding.

### 4. Implement

Implement the plan **one feature at a time, in the planned dependency order**, and within each feature
slice by slice — placing code in the project's feature-first structure and following its conventions.
Finish a feature (its slices, its tests, a green suite) before starting the next, and give the user a
short checkpoint between features so a long batch stays reviewable and can be paused. For changes that
touch existing code, work with a **safety net** (a git branch and commits per slice/feature, or a backup
if the project has no git — see `../refactor-application/references/execution.md`). Keep other features'
behavior unchanged — including features delivered earlier in the same batch. For any UI, follow
`../create-application/references/design.md` (professional UI/UX, no AI-slop tells) and match the
project's existing design language. For a real app, apply the secure-by-default baseline to the new code
(`../create-application/references/security.md`): boundary validation, authorization/ownership checks,
parameterized data access, SSRF/upload guards — matching the project's existing security patterns rather
than inventing parallel ones. Update the `README.md` and the tasks (`TaskUpdate`) as each feature lands.

### 5. Tests & validate

Follow the `test-application` methodology for each feature's tests:
- Detect the project's test setup (see `../test-application/references/detect-and-run.md`); reuse the
  existing framework or install the idiomatic one if missing.
- Write **real tests** for each feature at the right level (unit/integration/component/e2e — see
  `../create-application/references/testing.md`): happy path + relevant errors/edge cases. For a real
  app, include the security-relevant cases for what the feature added (see "Security testing" in
  `../create-application/references/security.md`): e.g. a denied cross-user/unauthenticated access on a
  new protected route, a rejected malformed/oversized payload.
- Run the affected tests as each feature lands, and the **full suite** after each feature and once more
  at the end; it must end green, plus lint, build and startup. Fix before moving on — never start the
  next feature on a red suite — and again before calling the batch done.
- **Manual-testing disclaimer**: tell the user which checks are still recommended to do manually,
  especially UI/UX interaction in a real browser, since graphical aspects are hard to verify reliably
  with AI.

Finish with a short report covering **each feature**: what was added, which files were created/changed,
the impact on existing features, the test results and the recommended manual checks.

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
