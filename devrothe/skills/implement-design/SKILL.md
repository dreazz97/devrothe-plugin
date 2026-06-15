---
name: implement-design
description: Redesigns an existing project's whole visual design to the user's requirements, acting as a design decision-support partner. Maps the project's current design system, then brainstorms the direction — asking about the app's theme/purpose, audience, mood/feeling, brand and inspiration to help the user pick the look (palette, typography, density, motion). Synthesizes a concrete design direction (light- and dark-mode palettes both checked for WCAG AA contrast) and a phased, behavior-preserving plan for approval; then reskins tokens-first with a safety net, keeping every feature working and both themes accessible. Use when the user wants to change or overhaul the look-and-feel, theme, color palette or design system of an existing app. Triggers on "implement-design", "redesign the app", "change the design", "new look and feel", "restyle", "new theme", "new color palette", and in Portuguese "muda o design", "redesenha a app", "novo visual", "nova paleta de cores", "muda o tema", "redesign da aplicação".
---

# implement-design

Take an existing project and **redesign its entire look-and-feel** to the user's requirements. This
skill is as much a **design decision-support partner** as an implementation skill: its distinctive job
is to help the user *decide* the direction — the theme, the feeling, the palette, the idea — and only
then apply it. Understand the current design first, brainstorm and converge on a direction, plan a
behavior-preserving reskin, then execute it through the design system.

**Rule: discover the direction before touching pixels.** The skill's core value is helping the user
choose, not guessing for them. Before any change, run the discovery (step 1): ask what the app *is* and
who it's for, the **mood/feeling** they want (e.g. calm, trustworthy, playful, premium, technical,
editorial), any brand constraints (existing logo/colors), and references they like — then surface
concrete options (palettes, type pairings, overall moods) with their trade-offs and **recommend** one.
Why: a redesign with no agreed direction is a reskin gamble; converging on the feeling first is what
makes the result coherent and *theirs* rather than a random new coat of paint.

**Rule: redesign the look, preserve the behavior.** Like a refactor, this changes the **visual layer**
— design tokens, styles, component appearance, and layout only where the plan says so — not what
features do, the data, the routes or the logic. Any deliberate UX or structural change (a re-laid-out
screen, a moved action) is a **named plan item**, never a silent side effect, and the existing security
posture must not be weakened (don't strip auth-gated UI, don't inline scripts/styles that break a CSP —
see `../create-application/references/security.md`). Why: the user asked for a new look; silently
changing functionality mid-reskin makes any regression impossible to attribute.

**Rule: tokens first, then components — drive it from the design system.** Define the new direction as
**design tokens** (palette, typography, spacing, radius, shadow, motion) in one central place and drive
the UI from them; reskin shared primitives/components so screens inherit the change, instead of spraying
one-off styles per screen. Why: a redesign applied as scattered per-screen overrides is inconsistent and
unmaintainable — single-source token control is the entire point of a design system, and it keeps the
change reviewable.

**Rule: accessible in every theme — mind light/dark contrast.** Meet **WCAG AA in both light and dark
mode** (≥4.5:1 for body text, ≥3:1 for large text and non-text UI like borders/icons/focus rings) — not
just one theme. Dark mode is **not the light palette inverted**: re-derive each token and re-check every
pair (text-on-background, text-on-accent, text-on-muted, text-on-elevated-surface) and every state
(hover/focus/disabled) in *both* themes. Why: a palette that passes in light routinely fails in dark
(and vice-versa), and a low-contrast dark theme is the single most common accessibility regression a
redesign introduces — see the contrast section in `references/design-direction.md`.

**Rule: professional UI/UX, not the generic AI look.** Apply
`../create-application/references/design.md` — UI/UX fundamentals (hierarchy, spacing scale, deliberate
type scale, focus/keyboard states, purposeful motion) and none of the AI-slop tells (emojis as UI,
cliché purple→blue gradients, untouched stock defaults, decoration overload, buzzword copy). If the
`frontend-design` skill is available, use it to produce the new UI. Why: a redesign is the moment to
*raise* the bar; shipping AI slop defeats the purpose.

**Rule: safety net + prove behavior unchanged.** A redesign touches many files, so branch/commit before
editing (gate — `../refactor-application/references/execution.md`) and, when done, the **existing test
suite must stay green** (proof the behavior was preserved) plus lint/build/startup, with a visual check
in **both themes**. Why: without a reversal path and a green suite there's no evidence you only changed
the look and not the app.

## Workflow

Copy this checklist into the response and tick items off:

```
- [ ] 0. Context — read CLAUDE.md, memory, READMEs/docs; map the current design system (styling approach, theme tokens, components, light/dark setup) and the app's screens; note the security posture
- [ ] 1. Discover — decision support: ask about the app's theme/purpose, audience, mood/feeling, brand and inspiration; surface palette/type options with trade-offs and help the user choose
- [ ] 2. Direction — synthesize a concrete design direction (palette incl. dark mode, typography, spacing/density, radius/shadow, motion, component style); verify WCAG AA contrast in BOTH themes; present for approval
- [ ] 3. Plan — map which tokens/components/screens change into a phased, behavior-preserving plan (TaskCreate); present for approval
- [ ] 4. Safety net + implement — branch/commit; reskin tokens-first → shared components → screens, incrementally; preserve behavior and security posture; keep both themes AA-accessible
- [ ] 5. Validate — existing tests stay green + lint/build/startup; verify both themes' contrast and states; manual UI/UX browser disclaimer
```

### 0. Project context

Before anything, read the metadata that explains the project and its conventions:
- `CLAUDE.md` at the root and in subfolders, and other agent instruction files (`AGENTS.md`,
  `.cursor/rules/`, `.github/copilot-instructions.md`).
- The project **memory** (design preferences and decisions already recorded).
- Text files: `README*`, `docs/`, `CONTRIBUTING.md`, design notes, brand guidelines.

Then **map the current design system** so the reskin works *with* it, not against it: the styling
approach (Tailwind theme, CSS variables, CSS modules, styled-components, plain CSS), where the theme
tokens live, the component library (shadcn/ui or other), how light/dark is wired (`next-themes`, a
`dark` class, a media query, or none yet), the global styles, and an inventory of the screens/components
the redesign will reach. Note the **security posture** too (PoC/demo vs real app — the gate in
`../create-application/references/security.md`) so the reskin doesn't weaken it. Full guidance in
`references/redesign-execution.md`. Why: the redesign must reskin the system that's actually there.

### 1. Discover the direction (decision support)

This is the heart of the skill. Use `AskUserQuestion` to brainstorm and converge — don't guess on
anything that shapes the look. Cover the app's **theme/purpose and industry**, its **audience**, the
**mood/feeling** they want (offer adjectives), **brand constraints** (existing logo, colors, fonts, or a
blank slate), **light/dark/both**, **density** (compact vs airy), **motion appetite**, and any
**references** they admire. Then **help them choose**: translate vague wants into a few concrete,
distinct directions (each with a name, mood, palette and type pairing) and **recommend** one with a
rationale — use option `preview`s for palette/type mockups when it helps the comparison. Full question
bank and the "help them decide" technique in `references/design-direction.md`.

### 2. Design direction

Synthesize the answers into **one concrete design direction** the user can approve (structure in
`references/design-direction.md`): the **color palette** as semantic tokens with **explicit light- and
dark-mode values**, the **typography** (≤2 families, a deliberate scale and weights), the **spacing
scale and density**, **radius/shadow/elevation**, **motion**, and the **component style**. Before
presenting, **verify WCAG AA contrast in both light and dark** for every text/surface pair and state
(see the contrast section). Present the direction — and, when useful, an alternative — for approval;
iterate steps 1–2 until the user is happy. Nothing is implemented yet.

### 3. Plan

Turn the approved direction into a **phased, behavior-preserving plan** (see
`references/redesign-execution.md`): the token layer first, then shared primitives/components, then
screen-by-screen, lowest-risk first. Map which tokens/components/screens each phase changes, and flag
any phase that involves a **deliberate layout/UX change** (not a pure reskin) so the user sees it
explicitly. Record the phases with `TaskCreate`. Present the plan for approval **before** editing.

### 4. Safety net + implement

Ensure a reversal path first (gate — `../refactor-application/references/execution.md`): a git branch
and a commit of the current state, or a backup if there's no git. Then execute the plan **tokens-first**:
update the central theme tokens (both light and dark), then reskin the shared primitives/components so
screens inherit the new look, then the screen-specific styles — phase by phase, committing per phase.
**Preserve behavior**: change styles/appearance and only the layout the plan named; don't touch logic,
routes, data, or the security posture. Keep **both themes AA-accessible** as you go (toggle and check,
don't leave dark for last). For any UI follow `../create-application/references/design.md`; reuse the
`frontend-design` skill if available. Update the `README.md`/design notes and the tasks (`TaskUpdate`)
as each phase lands.

### 5. Validate

Confirm the redesign changed only the look:
- Run the **existing test suite** — it must stay **green** (behavior preserved); fix any regression
  before continuing (detect/run via `../test-application/references/detect-and-run.md`). Add tests only
  if the plan deliberately changed behavior.
- **Contrast & states in both themes**: verify AA contrast, visible `focus-visible`, and
  hover/active/disabled/loading/empty/error states in **light and dark**; check responsiveness and
  `prefers-reduced-motion`.
- Confirm lint, build and startup pass.
- **Manual-testing disclaimer (always):** tell the user which checks need a real browser — the visual
  result, both themes side by side, responsiveness, animations and real flows — since graphical aspects
  are exactly what AI can't verify reliably, and a redesign is mostly graphical. List the concrete manual
  checks for the screens touched.

Finish with a short report: the chosen direction (before → after), the token/component/screen changes,
any deliberate layout changes and why, the test results, and the recommended manual/visual checks.

## References

- **`references/design-direction.md`** — the discovery interview (what to ask and how to help the user
  decide), building the design direction/brief as tokens, and the light/dark **WCAG AA contrast**
  guidance. Read in steps 1–2.
- **`references/redesign-execution.md`** — mapping the current design system, choosing the reskin
  strategy, the tokens-first behavior-preserving migration, and validating both themes. Read in steps
  0, 3–5.
- **Reused from other skills**: `../create-application/references/design.md` (professional UI/UX +
  anti-AI-slop, and the `frontend-design` skill); `../create-application/references/web-stack.md` /
  `app-stack.md` (where the Tailwind theme/shadcn tokens live);
  `../create-application/references/security.md` (the posture not to weaken);
  `../refactor-application/references/execution.md` (safety net + incremental execution);
  `../test-application/references/detect-and-run.md` (running the suite to prove behavior is preserved).
