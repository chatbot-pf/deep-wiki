---
description: Generate AGENTS.md files for pertinent repository folders — only where missing. Provides coding agents with build commands, testing instructions, code style, project structure, and boundaries.
disable-model-invocation: true
---

# Deep Wiki: Generate AGENTS.md Files

> **Note (Claude Code only):** This command is a thin entrypoint. The full procedure lives in the `wiki-agents-md` skill, which is the single source of truth and the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Invoke the **`wiki-agents-md`** skill to generate tailored `AGENTS.md` files (plus `CLAUDE.md` companions) for pertinent folders in this repository — follow its only-if-missing guard, pertinent-folder detection, six core areas, root/nested/wiki templates, validation, and generation report exactly. **Never overwrite an existing `AGENTS.md`.**

$ARGUMENTS
