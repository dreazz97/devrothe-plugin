# Safe execution and validation

How to apply the approved plan (restructure-plan.md) without losing work or breaking the application.

## Contents
- Safety net (gate)
- Execute phase by phase
- Preserve behavior
- Final validation and report

## Safety net (gate)

Before the first edit, ensure a reversal path:
- **Project with git**: confirm a clean working tree (or commit the current state) and create a working
  branch (e.g., `fix/restructure`). Commit at the end of each phase for a revertible history.
- **Project without git**: suggest `git init` + an initial commit; if the user refuses, make a backup
  copy of the folder before touching it.

Do not proceed to execution without one of these in place. Why: it is the only way to undo a refactor
that goes wrong.

## Execute phase by phase

- Apply one phase at a time, in the plan's order.
- At the end of each phase, **validate**: install deps, `lint`, `build`, tests and startup as the stack
  requires.
- Only move to the next phase with the current one green. Commit the phase (if git).
- Update the tasks (`TaskUpdate`) and the `README.md` as the state changes.
- If a phase introduces a hard-to-solve problem, **stop and report** to the user with the diagnosis,
  instead of forcing it or stacking workarounds.

## Preserve behavior

- Move/rename/reorganize and swap technologies while keeping the same functional result.
- The only allowed behavior changes are the **bugs** and the **behavior-changing security findings**
  listed in the plan — and both are recorded and approved (see the security findings section in
  `restructure-plan.md`). Do not alter security behavior silently any more than feature behavior.
- When a test exists for an area, use it as a net before and after refactoring it; if none exists and
  the area is risky, first create a test that pins the current behavior, then refactor.

## Final validation and report

When the phases are finished:
- Run the full test suite; **create tests** for the affected facets that did not have them (align with
  `../create-application/references/testing.md`), including the security-relevant tests for what was
  hardened (real app). The suite must end **green**.
- Confirm lint, build and startup with no errors.
- Present a final report: deviations fixed (before → after), bugs resolved, **security findings
  resolved** (with severities, and which fixes changed behavior) for a real app, files reorganized, and
  the test results (totals, failures, coverage if available). State the branch used for the user to
  review/integrate.
