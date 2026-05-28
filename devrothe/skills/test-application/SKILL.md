---
name: test-application
description: Validates, runs and (if missing) creates tests for the current application. Detects existing tests and maps coverage per facet. If everything is covered, it runs the tests and reports. If tests are missing — across all or only some facets (UI/frontend, backend/API, microservice, service, CLI/library, fullstack, mobile, data/ML) — it informs the user, proposes and (after approval) creates the missing tests and, at the end and with authorization, runs all of them (existing + created) and presents a report. Use when the user wants to run the tests, know whether the app has tests, validate the suite, or create tests. Triggers on "test-application", "run the tests", "does the app have tests?", "validate the tests", "create tests", "test coverage", and in Portuguese "testa a aplicação", "corre os testes", "a app tem testes?", "criar testes", "executar testes".
---

# test-application

Validate and run the current application's tests and, where they are missing, analyze the app, plan
and create real tests per facet, run them (with authorization) and present a report.

**Rule: real tests, not placeholders.** Created tests exercise real behavior (happy path +
errors/edge cases). Avoid trivial asserts, empty snapshots or mocks that return the expected result
without exercising the logic. Why: a green but empty suite gives false confidence.

## Flow

Copy this checklist into the response and tick items off. The branch is decided by **coverage per
facet**.

```
- [ ] 0. Context — read CLAUDE.md, memory and READMEs/docs of the project (if any)
- [ ] 1. Detect tests + analyze facets → coverage map per facet
- [ ] 2. Branch A (all facets covered) → run everything and report (step 8)
- [ ] 3. Branches B/C (missing coverage, total or partial) → report app type + facets with/without tests
- [ ] 4. Ask (single choice Yes/No) whether to create the missing tests
- [ ] 5. Test plan for the facets WITHOUT tests → user approval
- [ ] 6. Create the missing real tests
- [ ] 7. Ask (single choice Yes/No) whether it may run
- [ ] 8. Run ALL (existing + created) and present the report
```

### 0. Project context

Before detecting, look for and read metadata that helps understand the application:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** (preferences and decisions already recorded, including testing ones).
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, `ARCHITECTURE.md`, ADRs and notes.

Use this context to inform the facet analysis and the test plan/creation. Why: the README and the
CLAUDE.md usually state how to run the app and the tests and which conventions to follow.

### 1. Detect and map coverage

Detect test frameworks, scripts and files (see `references/detect-and-run.md`) **and** classify the
application's facets (see `references/app-analysis.md`). Cross-reference both to build a **coverage map
per facet**: for each present facet (UI, backend/API, microservice, etc.), does it have tests or not.
**Do not write or install anything in this phase** — only observe.

Decide the branch from the map:
- **All** facets have tests → **Branch A** (step 2).
- **No** facet has tests → **Branch B**.
- **Some** have, others do not → **Branch C** (partial coverage).

Branches B and C share steps 3–8. The only difference is the **scope of creation**: in Branch B tests
are created for all facets; in Branch C only for the facets **without** tests (the rest already exist
and are reused).

### 2. Branch A — everything covered

Run the full suite using the command defined by the project (prefer the project's script over
guessing) and proceed to step 8 (report). No authorization needed: the user invoked the skill to
validate and run.

If the run fails due to missing services (e.g., a database), point it out and suggest how to prepare
them (e.g., `docker compose up -d`) instead of marking the suite as broken.

### 3. Report (Branches B/C)

Present to the user, in a few lines: the **application type** and, from the coverage map, the facets
**with** tests (if any) and the facets **without** tests. Examples:
- Branch B: "Fullstack app (React + Vite and FastAPI/REST). No tests in any facet."
- Branch C: "Fullstack app: the backend/API has tests; the frontend does not."

### 4. Ask whether to create the missing tests

Use `AskUserQuestion` (single choice **Yes / No**) to ask whether tests should be created for the
missing facets.

If **No**:
- Branch C → run the **existing** tests and go to the report (step 8).
- Branch B → finish with a summary (app type + absence of tests + recommendation); there is nothing to
  run.

If **Yes** → continue to step 5.

### 5. Test plan

Create a plan **only for the facets without tests** (see `references/test-plan.md` for what to cover in
each). For each facet, list the targets/slices and the test type (unit, integration, component, e2e,
contract). Record them as tasks with `TaskCreate`. Present the plan and ask for approval **before**
writing any test.

### 6. Create the tests

Implement the approved plan's tests. Reuse the test framework already present in the project (in
Branch C there is almost always one); if missing, install the idiomatic one for the stack (see
`references/detect-and-run.md`) and configure it. Write real tests (see the rule above) and keep the
suite compiling.

### 7. Ask for authorization to run

Use `AskUserQuestion` (single choice **Yes / No**) to ask whether it may run the tests now.

If **No**: finish, stating the command(s) for the user to run the tests manually.
If **Yes**: proceed to step 8.

### 8. Report

Run **all** tests and present a report:
- Branch A → the existing suite.
- Branches B/C → **existing + just created** (in Branch C, both sets together).

```
# Test report
Command(s): <command(s) run>
Total: X · Passed: Y · Failed: Z · Skipped: W · Duration: <t>
Coverage: <%, if available>

## Failures
- <test> — <short reason>

## Conclusion
<green/red + suggested next steps>
```

When the just-created tests fail, distinguish in the report whether the failure is the test's or a real
application bug.

## References

- **`references/app-analysis.md`** — how to classify the application type and facets (signals to look
  for per ecosystem). Read in step 1.
- **`references/detect-and-run.md`** — detection of test frameworks/scripts/files and the commands to
  run per ecosystem; interpreting results. Read in steps 1, 2, 6 and 8.
- **`references/test-plan.md`** — what to test in each facet of the application and how to structure
  the plan. Read in steps 5 and 6.
