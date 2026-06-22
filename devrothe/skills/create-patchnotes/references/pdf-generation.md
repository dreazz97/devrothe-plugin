# PDF generation (deterministic, stack-independent)

Goal: turn the assembled HTML (the canonical template from `patchnotes-format.md`) into a PDF that comes
out the **same on any machine** and does **not** depend on the documented app's stack. The format lives
in the HTML/CSS; the engine is just a renderer. Pick the engine deterministically and verify the result.

## Contents
- Output path and naming
- Engine selection (in order)
- Install-on-demand
- Verify the result

## Output path and naming

- Directory: `docs/patch-notes/` if a `docs/` folder exists or it's reasonable to create one; otherwise
  the project root. Create the directory if missing.
- Base name: `<app-slug>-patch-notes-YYYY-MM-DD` (slug = the app name lowercased, spaces → hyphens; date
  = the implementation date).
- Write the assembled HTML to `<base>.html` first (it is the source of truth and is cheap to keep for
  re-rendering or a web view), then render `<base>.pdf` from it. Mention both paths in the final report;
  the user can delete the HTML if they only want the PDF.

## Engine selection (in order)

Use the **first** engine available. A Chromium-family renderer is the canonical choice — the template is
tuned for it and it gives identical output across machines — but the template's CSS is deliberately
print-safe so the fallbacks produce the same layout.

1. **Headless Chromium / Chrome / Edge** (canonical). Find a binary, then render:

   ```bash
   "<chrome-binary>" --headless=new --disable-gpu --no-pdf-header-footer \
     --print-to-pdf="<base>.pdf" "file://<absolute-path-to-base.html>"
   ```

   Binary lookup:
   - macOS: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome`,
     `/Applications/Chromium.app/Contents/MacOS/Chromium`,
     `/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge`.
   - Linux/PATH: `command -v google-chrome google-chrome-stable chromium chromium-browser microsoft-edge`.

   `--no-pdf-header-footer` suppresses Chrome's default date/URL header and footer (use
   `--print-to-pdf-no-header` on older builds) so only the template's own footer shows.

2. **Puppeteer** (if the project already has it, or Node is the project's ecosystem). Same Chromium
   engine → identical output. Launch, `page.setContent(html, { waitUntil: 'networkidle0' })` or load the
   `file://` URL, then `page.pdf({ path: '<base>.pdf', format: 'A4', printBackground: true })`.

3. **WeasyPrint** (`weasyprint <base>.html <base>.pdf`). Strong CSS support; renders the template
   faithfully. Needs its system libraries (pango/cairo) present.

4. **wkhtmltopdf** (`wkhtmltopdf --enable-local-file-access <base>.html <base>.pdf`). Older CSS engine —
   acceptable fallback; minor spacing differences are possible but the layout holds.

Do not invent a parallel format with a different tool (e.g. a from-scratch `reportlab`/`fpdf` script) —
that would break the "same format always" guarantee. Those generators are only acceptable if you
reproduce the canonical template's structure exactly, which is more work than installing a renderer.

## Install-on-demand

If no engine is found, do not silently produce nothing. Tell the user and offer to install one,
**preferring the project's ecosystem** and asking first (it touches the network and adds a dependency):
- Node project → `npm i -D puppeteer` (bundles its own Chromium; no system browser needed) — the most
  reliable cross-platform option.
- Python project → `pip install weasyprint` (note the system libs it needs).
- Otherwise → point the user at installing Chrome/Chromium, or installing one of the above in a throwaway
  environment.

Keep the install scoped and reversible; don't add a heavy dependency to the documented project's
manifest just to render a report unless the user agrees.

## Verify the result

After rendering, confirm `<base>.pdf` **exists and is non-empty** before reporting success (a renderer
can exit 0 yet write nothing if the input path was wrong). If it failed, report the engine and the error
rather than claiming a PDF was produced. Since the visual result of a document is hard to verify reliably
with AI, always tell the user to open the PDF and skim it as the final manual check.
