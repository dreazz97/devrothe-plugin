# Redesign execution

Read in steps 0 and 3–5. How to map the existing design system, plan the reskin, apply it tokens-first
without changing behavior, and validate that only the look changed — in both themes.

## Contents
- Map the current design system (step 0)
- Choose the reskin strategy
- The phased, behavior-preserving plan (step 3)
- Execute tokens-first (step 4)
- Validate both themes (step 5)

## Map the current design system (step 0)

Before planning, understand what you'll reskin so the change works *with* the existing system:

- **Styling approach** — Tailwind (with a theme in `tailwind.config`/CSS `@theme`), CSS variables, CSS
  modules, styled-components/emotion, vanilla CSS, or a mix. This decides where tokens live.
- **Token location** — the theme/design tokens (colors, radius, fonts): a Tailwind theme, a
  `globals.css` `:root`/`.dark` variable block, a shadcn/ui token set, or hard-coded per component.
  Centralized tokens make a clean reskin; hard-coded values mean you first consolidate them.
- **Component library** — shadcn/ui, a custom design-system folder, MUI/Chakra/etc., or ad-hoc. Shared
  primitives are where a token change propagates from.
- **Light/dark mechanism** — `next-themes`, a `dark` class toggled manually, a `prefers-color-scheme`
  media query, or none yet. If the user wants dark and there's no mechanism, adding one is a planned
  phase (and a behavior-adjacent change to flag).
- **Surface inventory** — the screens, layouts and shared components the redesign reaches, and the
  global styles (resets, base typography).
- **Security posture** — note it (`../create-application/references/security.md`) so the reskin doesn't
  weaken it (e.g. don't introduce inline styles/scripts that break a CSP, don't remove auth-gated UI).

## Choose the reskin strategy

- **In-place token swap (preferred)** — tokens are centralized; redefine them (and reskin a few
  primitives) and the UI inherits the new look. Lowest risk, most consistent.
- **Consolidate then reskin** — styles/colors are scattered as hard-coded values; first lift them into
  central tokens (a behavior-preserving refactor), then swap. Flag the consolidation as its own phase.
- **Deeper restyle / re-platform** — the user wants a structurally different UI or a styling-system
  change (e.g. adopting Tailwind + shadcn/ui from ad-hoc CSS). This is a bigger, riskier effort touching
  markup, not just styles — call it out explicitly, get approval, and align it with the target stack
  (`../create-application/references/web-stack.md` / `app-stack.md`). Prefer the lighter strategy when it
  achieves the goal.

## The phased, behavior-preserving plan (step 3)

Order phases lowest-risk first and so each leaves the app runnable:

1. **Token layer** — palette (light + dark), typography, spacing, radius, shadow, motion in the central
   theme. (If dark mode is new, wiring the mechanism goes here.)
2. **Shared primitives/components** — buttons, inputs, cards, nav, dialogs: reskin so screens inherit.
3. **Screens, lowest-risk first** — page by page, applying the new system.

For each phase note exactly which tokens/components/screens change. **Flag any deliberate layout/UX
change** (a re-laid-out screen, a moved action, a new dark-mode toggle) as its own named item — it is
*not* a pure reskin and the user must see it. Record phases with `TaskCreate`. Present for approval
before editing.

## Execute tokens-first (step 4)

- **Safety net first (gate)** — branch + commit the current state, or a backup if there's no git (see
  `../refactor-application/references/execution.md`). A redesign touches many files; without a reversal
  path there's no clean rollback.
- **Tokens, then primitives, then screens** — update the central tokens for **both light and dark**,
  then the shared components, then screen styles. Working from tokens keeps the change consistent and
  small at the source.
- **Preserve behavior** — change appearance and only the layout the plan named. Don't alter logic,
  routes, data, state, props/contracts or the security posture. Keep the DOM semantics and accessibility
  hooks (labels, roles, `aria-*`) intact — a reskin must not regress a11y.
- **Both themes as you go** — toggle light/dark after each phase and fix contrast/visual issues
  immediately; don't leave dark mode for the end (it's where regressions hide — see the contrast section
  in `references/design-direction.md`).
- **Commit per phase** and keep `README.md`/design notes and the tasks (`TaskUpdate`) in sync.
- Use `../create-application/references/design.md` for the UI/UX bar and the anti-AI-slop tells; reuse
  the `frontend-design` skill if available.

## Validate both themes (step 5)

- **Behavior preserved** — the **existing test suite stays green** (run it via
  `../test-application/references/detect-and-run.md`); a redesign should not turn the suite red. Add or
  update tests only for behavior the plan deliberately changed; update component snapshots that assert
  on styling intentionally.
- **Contrast & states in BOTH themes** — verify AA contrast, a visible `focus-visible` ring, and
  hover/active/disabled/loading/empty/error states in light *and* dark; check responsiveness (no
  horizontal scroll, mobile-first) and `prefers-reduced-motion`.
- **Build health** — lint, build and startup pass.
- **Manual/visual disclaimer (always)** — the visual outcome, the two themes side by side,
  responsiveness, animations and real flows need a **real browser**; AI can't verify graphical results
  reliably, and a redesign is almost entirely graphical. List the concrete checks per screen touched.

Report: the direction (before → after), the token/component/screen changes, any deliberate layout
changes and why, the test results, and the recommended manual/visual checks (both themes).
