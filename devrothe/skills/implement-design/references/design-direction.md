# Design direction (decision support)

Read in steps 1–2. This is the part that makes `implement-design` a *decision-support* skill: help the
user **decide** the look before any code changes. The flow is: run the discovery interview → translate
their answers into a few concrete directions → recommend one → lock it as a token-based design brief →
verify contrast in both themes. Never guess on a choice that shapes the look; never start coding a
direction the user hasn't approved.

## Contents
- The discovery interview (what to ask)
- Helping the user decide
- The design direction / brief (as tokens)
- Light & dark contrast (WCAG AA in both themes)

## The discovery interview (what to ask)

Use `AskUserQuestion`, batching related questions to reduce back-and-forth. Give each option a short
description so the choice is informed, and recommend where you have a basis to. Cover:

- **Theme / purpose & industry** — what the app is and the field it lives in (fintech, healthcare, dev
  tool, e-commerce, editorial/media, internal dashboard…). The domain carries visual expectations
  (a bank reads as trustworthy and restrained; a creative tool can be expressive).
- **Audience** — who uses it (consumers vs professionals, age, expertise, context: focused work vs
  casual browsing). Density, contrast and tone follow from this.
- **Mood / feeling** — the single most important question. Offer adjectives and let the user pick a few:
  *calm, trustworthy, premium/luxurious, playful, energetic, minimal, technical/precise, warm/friendly,
  editorial, bold, organic, retro, futuristic.* This is the "feeling" the design must deliver.
- **Brand constraints** — is there an existing logo, brand color, typeface or guideline to honor, or is
  it a blank slate? Capture hard constraints (a fixed brand color, an accessibility mandate) up front.
- **Color leanings** — any colors they love or refuse, and whether they want **light, dark, or both**
  (if both, dark is a first-class deliverable, not an afterthought — see the contrast section).
- **Density & layout feel** — compact/information-dense vs airy/spacious; how opinionated the layout
  should be.
- **Motion appetite** — none/subtle/expressive (always honoring `prefers-reduced-motion`).
- **References** — products or sites whose look they admire (and what specifically they like about
  each). Concrete references anchor an otherwise abstract conversation fast.

If the user can't answer the abstract questions, lead with references and mood — those unlock the rest.

## Helping the user decide

Don't dump the decision back on the user — **do the design thinking and present choices**:

- Translate vague wants into **2–3 distinct, named directions**, each with a one-line mood, a palette
  (a few hex anchors), a type pairing, and the trade-off (e.g. *"‘Editorial’ — warm paper neutrals +
  a serif display + a single ink accent; elegant and readable, less ‘app-like’"* vs *"‘Console’ —
  near-black surfaces, a mono/grotesk pair, one electric accent; technical and focused, harder to keep
  AA in light"*). Make them genuinely different, not three shades of the same idea.
- **Recommend one** with a rationale tied to their answers (mood + audience + brand), and say what
  you'd trade off. The user can pick, mix, or push back.
- Use option **`preview`s** (in `AskUserQuestion`) for palette swatches or type/scale mockups when a
  side-by-side visual makes the choice clearer than words.
- Iterate. A redesign direction is worth a round or two — converging here is cheaper than reworking the
  implemented UI.

## The design direction / brief (as tokens)

Once a direction is chosen, lock it as a **token-based brief** — the single source the implementation
will drive from (mapped to the project's system: Tailwind theme keys, CSS variables, or shadcn/ui
tokens; see `../create-application/references/web-stack.md` / `app-stack.md`). Specify:

- **Color — semantic tokens, with explicit light AND dark values** for each: `background`, `foreground`,
  `card`/elevated surface, `popover`, `muted` + `muted-foreground`, `border`, `input`, `ring` (focus),
  `primary` + `primary-foreground`, `secondary`, `accent`, plus state colors `destructive`, and (if
  used) `success`/`warning`/`info` — each with its readable foreground. Define color by **role**, not by
  raw hex sprinkled per component.
- **Typography** — at most ~2 families (a display/heading + a body, or one family across), a deliberate
  type scale, ≤3 weights, body ≥16px, line-height ~1.3–1.8, comfortable line length.
- **Spacing & density** — one spacing scale (4/8px steps) and the chosen density.
- **Radius, border, shadow/elevation** — consistent values (one radius scale, a small shadow set);
  decide how surfaces stack in both themes (in dark, elevation usually means *lighter* surfaces, not
  bigger shadows).
- **Motion** — durations/easings (~150–250ms, purposeful), honoring reduced-motion.
- **Iconography** — a single icon set (lucide-react), never emojis as UI.

Keep it restrained and intentional (a few deliberate choices beat a pile of effects — see
`../create-application/references/design.md`). The brief is what step 4 implements tokens-first.

## Light & dark contrast (WCAG AA in both themes)

If the design ships **light and dark**, contrast is verified in **both** — this is a hard gate, not a
nicety:

- **Targets**: ≥**4.5:1** for body text, ≥**3:1** for large text (~24px+/19px bold) and **non-text UI**
  (borders, icons, focus rings, input outlines, chart strokes) against their background.
- **Dark is re-derived, not inverted.** Flipping the light palette almost always breaks contrast and
  produces muddy or vibrating colors. Define dark values deliberately: dark surfaces are rarely pure
  black (use very dark neutrals), accents often need to be *lightened/desaturated* to stay readable on
  dark, and pure white body text on dark can be harsh — slightly off-white is calmer and still AA.
- **Check every pair, in both themes**: text-on-background, text-on-card/elevated, text-on-muted,
  **text-on-accent/primary** (the easiest to miss — a mid accent fails white *and* black text), and
  placeholder/disabled text (still legible, not invisible).
- **Check every state**: hover, active, **focus** (a visible `focus-visible` ring meeting 3:1),
  disabled, selected, and error/success text.
- **Don't rely on color alone** to convey meaning (error/success need an icon or text too).
- **Verify, don't eyeball.** Compute the ratios (any contrast tool / WCAG formula) for the token pairs;
  a token that fails AA in either theme is adjusted before it ships. A redesign that only checks the
  light theme is the most common way a dark mode lands inaccessible.
