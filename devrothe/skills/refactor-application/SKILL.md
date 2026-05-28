---
name: refactor-application
description: Analyzes an existing application and restructures it to follow the tech stack, fundamentals and organization of the create-application skill. Designed for projects built by inexperienced people: it identifies the app type and its facets, maps the deviations from the target methodology, detects bugs/errors and creates a phased restructuring plan (including the bug fixes). Once approved by the user, it executes the plan incrementally and behavior-preserving, and at the end validates with tests (runs the existing ones and creates the missing ones) plus lint/build/startup. Use when the user wants to align/normalize an existing project with the create-application practices, refactor the organization/stack, or "fix up" a poorly structured app. Triggers on "refactor-application", "refactor the application", "restructure the app", "align with create-application", "normalize the project", and in Portuguese "refatora a aplicação", "reestrutura a app", "normaliza o projeto", "arranja esta aplicação".
---

# refactor-application

Take an existing application — typically built by someone without experience — and restructure it to
the fundamentals, stack and organization of the `create-application` skill, fixing along the way the
bugs/errors that were already there.

**Rule: preserve behavior.** The restructuring does not change functionality — it is functional
equivalence (move, rename, reorganize, align technologies). The only exception are the bugs/errors,
which are fixed and must appear in the plan. Why: the user trusts that the app keeps doing the same;
changing features "on the side" of a refactor makes it impossible to know what broke.

**Rule: safety net before touching anything.** Do not change anything without a reversal path (see
step 4). Why: automated refactoring on someone else's code is risky; without history/backup there is
no way to roll back.

**Rule: incremental and validated.** Move phase by phase and validate (lint/build/tests/startup) at
the end of each one; only move to the next with the previous one green. Why: errors are caught early
and stay isolated to the phase that introduced them.

**Rule: professional UI/UX when aligning the frontend.** When the restructuring touches UI, bring it
up to `../create-application/references/design.md` — professional UI/UX fundamentals and none of the
AI-slop tells (emojis as UI, cliché purple→blue gradients, untouched stock defaults, buzzword copy).
Why: aligning to the target methodology includes the visual quality bar, not just the file layout.
Behavior preservation still holds — improve the design without changing what features do, and flag any
notable visual change as a deliberate plan item rather than a silent side effect.

**Target methodology.** The destination is defined by `create-application`. Consult:
`../create-application/references/web-stack.md` and `../create-application/references/app-stack.md`
(stack and folder structure), `../create-application/references/modules.md` (auth, storage, etc.),
`../create-application/references/testing.md` (tests) and `../create-application/references/design.md`
(UI/UX). To classify the app and detect/run tests, use `../test-application/references/app-analysis.md`
and `../test-application/references/detect-and-run.md`.

## Flow

Copy this checklist into the response and tick items off:

```
- [ ] 0. Context — read CLAUDE.md, memory and READMEs/docs of the project (if any)
- [ ] 1. Analysis — current stack, app type/facets, deviations from the target methodology and bugs/errors
- [ ] 2. Phased restructuring plan, including the fixes for detected bugs
- [ ] 3. Confirm the plan with the user (nothing is changed before the "yes")
- [ ] 4. Safety net — ensure git/branch or backup
- [ ] 5. Execute the plan — phase by phase, preserving behavior, updating the README
- [ ] 6. Validate — green tests (run existing; create missing) + lint/build/startup
```

### 0. Project context

Before analyzing, look for and read metadata that helps understand the application and the original
intent:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** (preferences and decisions already recorded).
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs and notes.

Use this context to inform the analysis, the deviation map and the restructuring plan. Why:
understanding what the app intends to do (and which conventions already exist) is essential to
restructure without changing behavior nor contradicting deliberate decisions.

### 1. Analysis

Analyze the project without changing it. Produce: the **application type** and facets, a **deviation
map** against the target methodology (language, tooling, UI, ORM, folder structure, tests, modules)
and the **list of existing bugs/errors** (broken build/type/lint, runtime, failing tests,
anti-patterns, security red flags). See `references/assessment.md`.

### 2. Restructuring plan

From the analysis, build a **phased** plan (low-risk first), preserving behavior. The plan must include
an explicit section of **bugs/errors to fix**. Record the phases/tasks with `TaskCreate`. See
`references/restructure-plan.md`.

### 3. Confirm

Present the full plan to the user: app type, deviations, bugs to fix, phases, effort and risks. Ask for
approval **before** changing anything. For high-effort migrations (e.g., framework swap), flag them and
offer phase-by-phase approval. Do not proceed without the "yes".

### 4. Safety net

Before any edit, ensure reversibility (gate — see `references/execution.md`): if the project uses git,
create a branch and commit the current state; if not, suggest `git init` + an initial commit or a
backup copy. Do not proceed without this.

### 5. Execute

Execute the plan phase by phase, following `references/execution.md`. In each phase: apply the changes,
preserve behavior, validate (lint/build/tests/startup) and update the `README.md` and the tasks
(`TaskUpdate`). If a phase breaks something hard to fix, stop and report instead of forcing it.

### 6. Validate

Confirm everything works: run the existing tests and **create the missing ones** for the affected
facets (align with `../create-application/references/testing.md`); the suite must end **green**, plus
lint, build and startup. Finish with a report of what changed, the bugs fixed and the test results.

**Manual testing disclaimer (always tell the user).** Include in the report that some checks are still
recommended to be done manually — especially **UI/UX interaction in a real browser** (visual layout,
responsiveness, animations, accessibility, real flows) — since graphical aspects are hard to verify
reliably with AI, and a refactor can shift visual behavior even when the suite stays green. List the
concrete manual checks for the areas touched (see `../create-application/references/testing.md`).

## References

- **`references/assessment.md`** — how to analyze the project: app type, deviation map against the
  target methodology and detection of bugs/errors. Read in step 1.
- **`references/restructure-plan.md`** — how to structure the phased plan (order, behavior
  preservation, bugs section). Read in step 2.
- **`references/execution.md`** — safe execution: safety net, phases, validation and report. Read in
  steps 4–6.
