# Patch-notes format (canonical)

Goal: every patch-notes PDF looks the **same** regardless of the application — same structure, sections,
typography and layout. Only the data changes. This file defines that structure, how to write each
feature, and the fixed HTML/CSS template to render.

## Contents
- Report structure
- Writing each feature (two audiences)
- The fixed template
- Placeholders and assembly
- Escaping

## Report structure

Top to bottom, always in this order:

1. **Header**
   - **Application name** — the large title at the top (human display name).
   - **Report title** — "Patch Notes" / "Release Notes" (localized to the report language).
   - **Implementation date** — the date the features were implemented (absolute, friendly format).
   - **Version** — optional, only if the project has one (tag, `package.json` version, etc.).
2. **Release summary** — optional single line introducing the release ("This release adds X and improves
   Y."). Skip it if it would just restate the list.
3. **Feature list** — one entry per feature, in a sensible reading order (most significant first, or
   grouped New → Improvements → Fixes). Each entry:
   - **Title** — short and human.
   - **Tag** — New / Improvement / Fix.
   - **In short** — the plain-language summary.
   - **What was done** — the technical note.
4. **Footer** — generation date and the application name, once at the end of the document.

Keep the report self-contained and factual: no marketing fluff, no emojis as decoration, no invented
metrics. Hold it to the professional bar in `../../create-application/references/design.md`.

## Writing each feature (two audiences)

Each feature is written **twice**, for two readerships — this is the core requirement:

- **In short** (for everyone) — 1–2 sentences, plain language, **no jargon and no file/library names**.
  Say what the feature lets a person do and why it matters. A non-technical team member should fully
  understand it.
  - Good: *"Users can now reset their own password by email, without contacting support."*
  - Bad: *"Added a `POST /auth/reset` endpoint backed by a signed JWT and a Resend transport."*

- **What was done** (technical) — the implementation detail, in full sentences a careful non-developer
  can still follow: the areas/modules/endpoints/components added or changed, key libraries introduced,
  any data model/migration, and the behavior added. Technical enough to be useful to a developer; expand
  an acronym on first use.
  - Example: *"Added a password-reset flow: a new `POST /auth/reset` endpoint issues a single-use,
    time-limited token, the email is sent through the existing email provider, and a new reset page
    validates the token before accepting a new password. No database migration was needed."*

Derive the technical note from the conversation and the actual commits/diff — do not guess. If the
evidence doesn't support a claim, leave it out.

## The fixed template

Render this exact template (placeholders in `{{...}}`). It uses only print-safe, widely-supported CSS
(system fonts, simple flow layout, a single solid accent — no gradients) so the output is **identical
across rendering engines**. Do not restyle per project.

```html
<!DOCTYPE html>
<html lang="{{LANG}}">
<head>
<meta charset="utf-8">
<title>{{APP_NAME}} — {{REPORT_TITLE}}</title>
<style>
  :root {
    --ink: #18212f;
    --muted: #5b6573;
    --line: #e3e7ee;
    --accent: #1f4e8c;        /* solid, professional — never a gradient */
    --bg-soft: #f6f8fb;
    --tag-new: #1f7a4d;
    --tag-improvement: #1f4e8c;
    --tag-fix: #9a5b00;
  }
  @page { size: A4; margin: 18mm 16mm; }
  * { box-sizing: border-box; }
  html { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
  body {
    margin: 0;
    color: var(--ink);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    font-size: 10.5pt;
    line-height: 1.5;
  }
  header.cover {
    border-bottom: 3px solid var(--accent);
    padding-bottom: 14px;
    margin-bottom: 22px;
  }
  .app-name { font-size: 26pt; font-weight: 700; letter-spacing: -0.01em; margin: 0; }
  .report-title { font-size: 12pt; font-weight: 600; color: var(--accent); margin: 4px 0 0; text-transform: uppercase; letter-spacing: 0.08em; }
  .meta { color: var(--muted); font-size: 10pt; margin-top: 10px; }
  .meta strong { color: var(--ink); font-weight: 600; }
  .summary { font-size: 11pt; color: var(--ink); margin: 0 0 22px; }
  .feature {
    break-inside: avoid;
    page-break-inside: avoid;
    border: 1px solid var(--line);
    border-left: 4px solid var(--accent);
    border-radius: 6px;
    background: #fff;
    padding: 14px 16px;
    margin-bottom: 14px;
  }
  .feature-head { display: flex; align-items: baseline; gap: 10px; margin-bottom: 8px; }
  .feature-title { font-size: 13pt; font-weight: 700; margin: 0; }
  .tag {
    font-size: 8pt; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em;
    padding: 2px 8px; border-radius: 999px; color: #fff; white-space: nowrap;
  }
  .tag.new { background: var(--tag-new); }
  .tag.improvement { background: var(--tag-improvement); }
  .tag.fix { background: var(--tag-fix); }
  .block-label { font-size: 8.5pt; font-weight: 700; text-transform: uppercase; letter-spacing: 0.06em; color: var(--muted); margin: 10px 0 2px; }
  .in-short { margin: 0; }
  .technical { margin: 0; color: #2b3645; background: var(--bg-soft); border-radius: 4px; padding: 8px 10px; }
  footer.doc-footer { margin-top: 26px; padding-top: 10px; border-top: 1px solid var(--line); color: var(--muted); font-size: 9pt; display: flex; justify-content: space-between; }
</style>
</head>
<body>
  <header class="cover">
    <h1 class="app-name">{{APP_NAME}}</h1>
    <p class="report-title">{{REPORT_TITLE}}</p>
    <p class="meta"><strong>{{LABEL_DATE}}:</strong> {{IMPLEMENTATION_DATE}}{{VERSION_BLOCK}} · <strong>{{FEATURE_COUNT}}</strong> {{LABEL_FEATURES}}</p>
  </header>

  {{SUMMARY_BLOCK}}

  <main>
    {{FEATURES}}
  </main>

  <footer class="doc-footer">
    <span>{{APP_NAME}} — {{REPORT_TITLE}}</span>
    <span>{{LABEL_GENERATED}}: {{GENERATED_DATE}}</span>
  </footer>
</body>
</html>
```

Per-feature block (one per feature, assembled into `{{FEATURES}}`):

```html
<section class="feature">
  <div class="feature-head">
    <h2 class="feature-title">{{FEATURE_TITLE}}</h2>
    <span class="tag {{TAG_CLASS}}">{{TAG_LABEL}}</span>
  </div>
  <p class="block-label">{{LABEL_IN_SHORT}}</p>
  <p class="in-short">{{FEATURE_IN_SHORT}}</p>
  <p class="block-label">{{LABEL_WHAT_WAS_DONE}}</p>
  <p class="technical">{{FEATURE_TECHNICAL}}</p>
</section>
```

## Placeholders and assembly

| Placeholder | Value |
|---|---|
| `{{LANG}}` | BCP-47 code of the report language (`en`, `pt`, …) |
| `{{APP_NAME}}` | Application display name |
| `{{REPORT_TITLE}}` | "Patch Notes" / "Release Notes" (localized) |
| `{{IMPLEMENTATION_DATE}}` | Absolute, friendly date (e.g. `22 June 2026`) |
| `{{VERSION_BLOCK}}` | ` · <strong>vX.Y.Z</strong>` if there is a version, else empty |
| `{{FEATURE_COUNT}}` | Number of features |
| `{{SUMMARY_BLOCK}}` | `<p class="summary">…</p>` if there is a release summary, else empty |
| `{{FEATURES}}` | The concatenated per-feature blocks |
| `{{GENERATED_DATE}}` | The date the PDF was generated |
| `{{TAG_CLASS}}` | `new` / `improvement` / `fix` |
| `{{TAG_LABEL}}` | The localized tag label (New / Improvement / Fix) |

Localize the **label** placeholders to the report language, keeping the structure identical:
`{{LABEL_DATE}}` (Date), `{{LABEL_FEATURES}}` (features), `{{LABEL_GENERATED}}` (Generated),
`{{LABEL_IN_SHORT}}` (In short), `{{LABEL_WHAT_WAS_DONE}}` (What was done).

Optional Chrome-only enhancement: per-page page numbers via `@page` margin boxes. Leave it **off by
default** — it renders differently across engines and would break the "identical output" guarantee. The
canonical footer is the single end-of-document one above.

## Escaping

Every value injected into the HTML (titles, summaries, technical notes, app name) must have `&`, `<` and
`>` escaped to `&amp;`, `&lt;`, `&gt;` so feature text containing code (`<Button>`, `a && b`) does not
break the markup.
