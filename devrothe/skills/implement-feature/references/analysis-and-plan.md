# Codebase analysis and plan

Goal: understand exactly where the feature fits before building it, and turn that into an approvable
plan. Always produce a plan; run the extensive analysis below when the conversation does not already
pin down the feature and the affected code.

## Contents
- When to analyze deeply
- What to map
- Plan structure

## When to analyze deeply

- **Enough context already** (the conversation specifies the feature and the area, and the codebase is
  small/known) → a light plan is fine; still write it down for approval.
- **Not enough context** → run the full analysis below before planning. Do not start coding to "figure
  it out"; the plan is cheaper to change than code.

## What to map

Read the relevant code (do not change it yet) and determine:

- **Where it fits** — which feature/module folder it belongs to in the existing structure; whether a
  new feature folder is warranted (align with `../create-application/references/web-stack.md` /
  `app-stack.md`).
- **Data model** — new entities/fields, and whether a migration is needed (and if it is destructive).
- **Routes/endpoints** — new routes/handlers, or changes to existing ones (watch for contract changes).
- **Shared code it uses or changes** — utilities, components, hooks, services, types; reusing is good,
  modifying shared code is a blast-radius signal (feed it into `impact-analysis.md`).
- **Integration points** — auth/permissions, storage, payments, email, external APIs, background jobs
  (see `../create-application/references/modules.md`).
- **Conventions to follow** — patterns from CLAUDE.md, existing similar features, naming, error
  handling, validation (Zod/Pydantic).

## Plan structure

Present, before asking for approval:
- **Summary** — what the feature does and where it lives.
- **Slices** — small vertical slices ordered by dependency, each with: what it adds, files to
  create vs touch, and the tests that cover it.
- **New dependencies / migrations** — call them out explicitly.
- **Docs** — the README/docs updates the feature will require.

Record the slices as tasks with `TaskCreate`. The plan must be enough for the user to approve the
"what" before any code is written; the blast-radius "risk" is stated separately in step 3.
