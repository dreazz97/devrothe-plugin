# Devrothe plugin — contributor conventions

This repository is a Claude Code plugin (a marketplace + the `devrothe` plugin with its skills). Read
this before adding or changing anything.

## Language convention (always apply)

**All plugin artifacts are authored in English** — SKILL.md bodies, reference files, README, manifest
descriptions and code comments. This holds **regardless of the language used in the conversation**:
even when the user writes in Portuguese, every new or modified skill, reference and doc is written in
English.

**Skill descriptions must keep bilingual triggers (English + Portuguese).** The `description` field is
what makes a skill fire, so it must list trigger phrases in both English and Portuguese so the skill is
discovered whether the user prompts in EN or PT. Keep the descriptive part in English; only the trigger
phrases are duplicated across languages.

Note: this rule governs the **artifacts**, not the conversation. Reply to the user in the language they
use (e.g., Portuguese) — but write the files in English.

## Skill authoring conventions

Follow Anthropic's official Agent Skills best practices, consistent with the existing skills:

- **Frontmatter**: `name` (lowercase, hyphens) and a third-person, "pushy" `description` (≤ 1024 chars)
  that states what the skill does and when to use it, ending with the bilingual triggers.
- **Progressive disclosure**: keep `SKILL.md` lean (well under 500 lines); push detail into
  `references/*.md`, kept **one level deep** from SKILL.md.
- **Imperative voice** in instructions; **explain the why** behind each rule so the model can
  generalize to unanticipated cases.
- **Naming**: skills use the `verb-application` family (`create-application`, `test-application`,
  `refactor-application`).
- **Context first**: every skill starts with a step that reads existing project metadata (CLAUDE.md,
  memory, READMEs/docs) before acting.
- Preserve code blocks, file paths and cross-skill references (`../<skill>/references/...`) when
  editing.

## After changing the plugin

- Validate JSON manifests and keep this README/manifests in sync with the skills.
- Bump `version` in `devrothe/.claude-plugin/plugin.json`.
- Reload locally with `/plugin marketplace update devrothe`.
