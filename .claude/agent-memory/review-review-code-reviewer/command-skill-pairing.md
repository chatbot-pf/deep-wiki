---
name: command-skill-pairing
description: deep-wiki plugin convention — skills/*/SKILL.md is the single source of truth; commands/*.md are thin delegating aliases
metadata:
  type: project
---

The deep-wiki plugin pairs each `commands/<x>.md` with a `skills/wiki-<x>/SKILL.md`.

**Convention:**
- Skills hold all substantive procedure and carry frontmatter `user-invocable: false`. Required frontmatter: `name`, `description` (both present across all skills).
- Commands are thin entrypoints: frontmatter `disable-model-invocation: true`, body is a short "Invoke the `<skill>` skill ..." delegation ending in `$ARGUMENTS`.
- The two invocation paths do not conflict: `disable-model-invocation` (command) stops model auto-invoking the alias; `user-invocable: false` (skill) stops users invoking the skill directly. User reaches the skill via the command; model reaches it via the skill.

**Build pair exception:** the large VitePress spec lives in `skills/wiki-vitepress/references/vitepress-build.md` (a separate reference file); the SKILL.md is a concise overview that links to it via relative `references/vitepress-build.md`. `commands/build.md` links to it via `../skills/wiki-vitepress/references/vitepress-build.md` (commands/ and skills/ are siblings).

**How to apply:** When reviewing changes here, verify (a) folded content actually landed in the target skill (don't assume), (b) relative reference links resolve given the sibling commands//skills/ layout, (c) the frontmatter pair is present on both sides.
