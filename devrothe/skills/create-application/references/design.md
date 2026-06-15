# UI/UX design rules

Read when building any UI (Next.js or React+Vite). Two goals: a **professional, accessible** interface
and **avoiding the generic "AI-generated" look**. If the `frontend-design` skill is available, use it
to produce the UI; otherwise apply the rules below.

## Contents
- Professional UI/UX baseline
- Avoid the AI-generated look
- Quick checklist

## Professional UI/UX baseline

- **Visual hierarchy**: guide the eye with size, weight, contrast, spacing and position. One clear
  primary action per view; secondary actions are visually quieter.
- **Spacing & rhythm**: use one consistent spacing scale (4/8px steps), align to a grid, group related
  elements (proximity) and give the layout generous, intentional whitespace.
- **Typography**: a deliberate type scale; at most ~2 families and ≤3 weights; body ≥16px; line-height
  ~1.3–1.8; line length ~60–75 chars; left-align body text — do not center long paragraphs.
- **Color**: a restrained palette with intent (neutrals + ~1 accent). Avoid pure black on pure white.
  Meet WCAG AA contrast (≥4.5:1 for body text, ≥3:1 for large text and UI elements).
- **Light & dark mode**: when the app ships both themes (the default here — `next-themes`), each is a
  first-class design, not an afterthought, and every component must stay legible in both. Dark mode is
  **re-derived, not the light palette inverted**: define each token's dark value deliberately (dark
  surfaces are very-dark neutrals, not pure black; accents usually need lightening/desaturating to stay
  readable; near-white body text beats harsh pure white). **Verify WCAG AA contrast in BOTH themes** for
  every text/surface pair — including text-on-accent, text-on-muted and text-on-elevated-surface — and
  every state (hover/focus/disabled), since a palette that passes in one theme routinely fails in the
  other. Non-text UI (borders, icons, focus rings) needs ≥3:1 in both. Verify the ratios, don't eyeball
  them.
- **Consistency**: reuse components and design tokens (shadcn/ui + the Tailwind theme); keep radii,
  shadows and borders consistent — do not sprinkle arbitrary values.
- **States & feedback**: every interactive element has hover/focus/active/disabled, and views have
  loading/empty/error states. Keep a visible keyboard focus (`focus-visible`).
- **Accessibility**: semantic HTML, labels, alt text, full keyboard navigation, target sizes ≥24–44px,
  and respect `prefers-reduced-motion`.
- **Responsive**: design mobile-first, verify breakpoints, no horizontal scroll.
- **Motion (framer-motion)**: purposeful and subtle (~150–250ms); do not animate everything; honor
  reduced-motion.
- **Content**: real, specific copy and data — never ship lorem ipsum or placeholder text in the final
  UI.

## Avoid the AI-generated look

These tells make a UI look mass-produced and unprofessional — avoid them:

- **Emojis as UI** — no emojis in headings, buttons, navigation, bullet lists or as feature "icons".
  Use a proper icon set (lucide-react). This is the most obvious tell.
- **Cliché gradients** — avoid the default purple→blue/indigo hero gradient and gradient text
  everywhere; use color with intent.
- **Boilerplate layout** — the "centered hero + three identical feature cards + testimonials + pricing"
  template applied blindly. Shape the layout around the actual content.
- **Untouched defaults** — do not ship the stock Inter/Poppins + slate palette + uniform rounded-card
  look. Customize the theme (a distinctive typeface, scale and accent).
- **Filler copy & buzzwords** — avoid "unlock", "elevate", "seamless", "supercharge", "leverage",
  "robust", "drive meaningful results", and the "[adjective] solution that helps you [verb]" formula.
  Write concrete, specific copy.
- **Decoration overload** — glassmorphism, neon glows, blur blobs, drop shadows on everything, gradient
  borders. Use sparingly or not at all.
- **Fake content** — no invented testimonials, logos or team members; use real data or clearly marked
  placeholders.

Default to **restraint and intention**: a few deliberate choices (type, spacing, one accent) beat a
pile of effects. Why: the generic AI aesthetic is instantly recognizable and reads as low-effort;
crafted, consistent design is the professional baseline.

## Quick checklist

```
- [ ] No emojis in the UI; icons via lucide-react
- [ ] Consistent spacing scale and grid alignment
- [ ] Deliberate type scale, ≤3 weights, body ≥16px, readable line-height/length
- [ ] WCAG AA contrast; no pure black on pure white
- [ ] Contrast met in BOTH light and dark themes (dark re-derived, not inverted); states checked in both
- [ ] Visible focus states; full keyboard navigation
- [ ] One clear primary action per view
- [ ] Real copy/data — no lorem ipsum or buzzword filler
- [ ] No cliché purple→blue gradient or blindly-applied boilerplate hero
- [ ] Responsive, mobile-first; prefers-reduced-motion respected
```
