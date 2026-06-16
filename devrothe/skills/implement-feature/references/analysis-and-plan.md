# Codebase analysis and plan

Goal: understand exactly where the feature(s) fit before building them, and turn that into an approvable
plan. The request may be a single feature or a list — analyze each, but produce **one** consolidated
plan. Always produce a plan; run the extensive analysis below when the conversation does not already pin
down the feature(s) and the affected code.

## Contents
- When to analyze deeply
- What to map
- Ordering several features
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
- **Integration points** — auth/permissions, storage, payments, email (SMTP/Graph/Resend; real-app
  creds in the encrypted vault — DB + Fernet), external APIs, background jobs
  (see `../create-application/references/modules.md`).
- **Conventions to follow** — patterns from CLAUDE.md, existing similar features, naming, error
  handling, validation (Zod/Pydantic).
- **Inter-feature dependencies** (when several features were requested) — which feature must exist
  before another can be built, and which features touch the same shared code, data model or routes.
  This drives both the build order and the cross-feature impact (feed it into `impact-analysis.md`).

## Ordering several features

When the request is a list, decide the build order before writing the plan:

- **Dependency first** — a feature others build on (a shared model, a base component, an auth change)
  goes before the features that consume it.
- **Group overlapping features** — features that edit the same files are easier to reason about, and to
  review, when planned and built next to each other.
- **Keep each feature independently shippable** — order so that finishing any prefix of the list leaves
  the app green and usable, in case the user pauses the batch.

## Plan structure

Present, before asking for approval:
- **Summary** — what each feature does and where it lives; for a list, the order they will be built and
  why.
- **Per feature** — a section per feature, each with its **slices**: small vertical slices ordered by
  dependency, each with what it adds, files to create vs touch, and the tests that cover it. (For a
  single feature this is just one section.)
- **New dependencies / migrations** — call them out explicitly (note which feature needs each).
- **Docs** — the README/docs updates the feature(s) will require.

Record the slices as tasks with `TaskCreate`, one task group per feature. The plan must be enough for the
user to approve the "what" and the order before any code is written; the blast-radius "risk" is stated
separately in step 3.
