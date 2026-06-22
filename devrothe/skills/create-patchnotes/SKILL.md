---
name: create-patchnotes
description: Generates a fixed-format PDF patch-notes (release-notes) report for the features just implemented in the current application. Identifies which features to include by reading the current conversation first, then corroborating against the recent git commits, and — on any doubt or when several candidates exist — asks the user to pick from the available list. Reads the app name and implementation date, and writes each feature both in plain language for non-technical readers and with enough technical detail of what was done. Renders a consistent, professional PDF layout that is identical regardless of the application or its stack. Use when the user wants a patch-notes / release-notes PDF of recently added features. Triggers on "create-patchnotes", "patch notes", "release notes", "changelog PDF", "generate patch notes", "report of the features", and in Portuguese "cria as patch notes", "notas de versão", "notas de lançamento", "relatório das features", "PDF das funcionalidades".
---

# create-patchnotes

Produce a **PDF patch-notes report** for the features that were just implemented in the current
application. The report always has the **same layout** regardless of the app: the application name at
the top, the implementation date, and a list of the features — each described **twice over**: a
plain-language summary anyone can understand and a technical note of what was actually done. This skill
pairs naturally after `implement-feature` or `create-application`, but it is standalone: it figures out
the feature list itself and only generates the document.

**Rule: identify the features by source precedence — conversation → commits → ask.** Decide what goes
in the patch notes in this order: (1) the **current conversation** — the features built in this session
(richest source: it has the intent and the detail); (2) corroborate and fill gaps from the **recent git
commits** (messages + diffstat tell you which features landed and what changed); (3) **on any doubt** —
the source is unclear, candidates overlap, or there are several possible features — use
`AskUserQuestion` to let the user pick from the available list. Why: the conversation knows *why* a
feature was built, the commits are the objective record of *what* changed, and the user is the
authority when those two don't pin the scope down.

**Rule: the format is fixed and identical for every application.** Always render the **canonical
template** in `references/patchnotes-format.md` — same structure, sections, typography and layout — only
the data (app name, date, feature entries) changes. Do not depend on the project's own stack or design
to build the report. Why: patch notes are a recurring deliverable; a stable, recognizable format is the
point, so a reader knows exactly where to look every time.

**Rule: write each feature for two audiences.** Every feature entry has a plain-language summary (no
jargon, no file names — what it does and why it matters to someone using the app) **and** a technical
note (the areas/modules/endpoints/components changed, key libraries, data/migrations, behavior added —
technical enough to be useful to a developer, but in full sentences a careful non-developer can still
follow). Why: the user asked for a report that serves both technical and non-technical team members, so
neither audience is left guessing.

**Rule: it is an outward-facing deliverable — confirm the content before rendering.** Assemble the
patch notes (app name, date, per-feature text) and present it for the user's approval **before**
producing the PDF. Why: the wording is read by other people and the file is shared; getting the scope
and phrasing right up front is cheaper than re-cutting a document that already went out.

**Rule: generate the PDF reliably, not via the app's stack.** Use the deterministic engine selection in
`references/pdf-generation.md` (a Chromium-family renderer first, documented fallbacks otherwise) so the
output is consistent across machines and does not assume the project is Node, Python or anything else.
Why: the report must come out the same whether the app being documented is a Rust CLI or a Next.js site.

## Workflow

Copy this checklist into the response and tick items off:

```
- [ ] 0. Context — read CLAUDE.md, memory, READMEs/docs; resolve the app name, the language to write in, and the date
- [ ] 1. Gather features — from the conversation first, then corroborate with recent git commits; assemble the candidate list with "what was done"
- [ ] 2. Resolve doubts — if the source is unclear, features overlap, or there are several candidates, ask the user to pick from the available list (AskUserQuestion)
- [ ] 3. Compose — write each feature for both audiences (plain-language + technical); present the assembled patch notes for approval
- [ ] 4. Render — assemble the fixed HTML template with the data and produce the PDF with a reliable engine
- [ ] 5. Report — give the user the output path and the recommended manual check (open the PDF)
```

### 0. Context

Before anything, read the metadata that identifies the application and how to write about it:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** (decisions and preferences already recorded).
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, and the manifests (`package.json`, `pyproject.toml`,
  `composer.json`, `Cargo.toml`, …).

Resolve three things here:
- **Application name** — prefer a human display name (README H1, a `displayName`/`title`, CLAUDE.md
  title) over a package slug; keep a slug form for the filename. Ask the user if there is none or it is
  ambiguous.
- **Language of the report** — patch notes are an **end-user deliverable**, not a plugin artifact, so
  write the content in the language the user is working in (default to the conversation's language; ask
  if unsure). The *layout* stays the canonical one either way — only the wording is localized.
- **Implementation date** — default to **today** (the features were just implemented); if the features
  come from older commits, use the commit date or a date range. Convert any relative date to an absolute
  one. Confirm with the user when it is not obvious.

### 1. Gather features

Build the candidate list following the source precedence (see the rule above):

1. **Conversation** — list the features implemented in this session, with the detail you already have
   (intent, the areas touched, the libraries added, the behavior). This is the primary source.
2. **Recent git commits** — corroborate and discover anything the conversation missed. Inspect the
   relevant window of history and read both the messages and the changes:
   - Decide the window: the commits made during this session, or since the last tag/release, or the last
     handful — if it is not obvious, confirm the range with the user.
   - `git log --stat` (or `git show --stat <ref>`) to see which features landed and which files/areas
     each one touched; use the diff to ground the **technical** "what was done", not to guess.
   - Map commits to user-facing features (group related commits; ignore pure chore/format/CI commits
     unless they are themselves the change worth noting).

For each candidate, capture: a human **title**, a **tag** (New / Improvement / Fix), the **plain-language
summary**, and the **technical note** (areas/modules/endpoints/components, key libraries,
data/migrations, behavior). Don't fabricate detail — if the conversation and the commits don't support a
claim, leave it out or ask.

### 2. Resolve doubts

If after step 1 the scope is still unclear — the source disagrees, candidates overlap, there are more
features than obviously belong in this release, or you simply can't tell which ones the user means — use
`AskUserQuestion` (multi-select) to present the **available list** and let the user choose which features
go into the patch notes, and confirm the implementation date and app name if those were uncertain too.
Skip this step when the conversation already makes the set unambiguous. Don't guess the scope of a
document other people will read.

### 3. Compose and confirm

Write the report content following `references/patchnotes-format.md`:
- The **header** data: app name, report title, implementation date (and a version if the project has
  one), and an optional one-line release summary.
- One **entry per feature**, each with its plain-language summary and its technical note, written for
  the two audiences (see the rule). Keep titles human ("Email notifications", not `feat: add SMTP
  provider`); keep file names and library names out of the plain-language part and inside the technical
  note. Hold the writing to the professional bar in `../create-application/references/design.md` — clear,
  specific, no buzzword filler, no emojis as decoration.

Present the assembled patch notes (as text) and get the user's **approval before rendering**. Iterate on
the wording until they're happy.

### 4. Render

Assemble the canonical HTML template (`references/patchnotes-format.md`) with the approved data —
escaping `&`, `<`, `>` in any injected text — and produce the PDF using the engine selection in
`references/pdf-generation.md`. Write the output to a predictable, dated path
(`<app-slug>-patch-notes-YYYY-MM-DD.pdf`, see the reference for the directory and naming). Verify the PDF
file was actually created (non-empty) before reporting success; if no rendering engine is available,
tell the user and offer to install one (preferring the project's ecosystem) rather than silently
producing nothing.

### 5. Report

Tell the user:
- the **output path** of the PDF (and the source HTML kept alongside it),
- the **features included** and the date/app name used,
- the engine that rendered it,
- and the **manual check**: open the PDF to confirm the layout and the wording read well — the visual
  result of a document is hard to verify reliably with AI.

## References

- **`references/patchnotes-format.md`** — the canonical report format: structure, how to write each
  feature for both audiences, and the fixed HTML/CSS template with its placeholders. Read in steps 3
  and 4.
- **`references/pdf-generation.md`** — deterministic, stack-independent PDF generation: engine detection
  and commands, install-on-demand, output path/naming and verification. Read in step 4.
- **`../create-application/references/design.md`** — the professional UI/UX / anti-AI-slop bar the
  template and the writing follow. Read in step 3.
