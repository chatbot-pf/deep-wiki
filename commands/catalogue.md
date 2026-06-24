---
description: Generate only the hierarchical wiki structure (table of contents) as JSON for the current repository
disable-model-invocation: true
---

# Deep Wiki: Catalogue Generation

> **Note (Claude Code only):** This command is a thin entrypoint. The full procedure lives in the `wiki-architect` skill, which is the single source of truth and the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Invoke the **`wiki-architect`** skill to generate **only** the hierarchical JSON catalogue (table of contents) for this repository — follow its source-repository resolution, JSON schema, and constraints exactly. Output the JSON catalogue only (catalogue-only mode); do not generate page content.

$ARGUMENTS
