---
description: Generate a single wiki page with dark-mode Mermaid diagrams, source citations, and first-principles depth
argument-hint: [topic] [optional file paths]
disable-model-invocation: true
---

# Deep Wiki: Single Page Generation

> **Note (Claude Code only):** This command is a thin entrypoint. The full procedure lives in the `wiki-page-writer` skill, which is the single source of truth and the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Invoke the **`wiki-page-writer`** skill to generate a comprehensive wiki page for the topic below — follow its source-repository resolution, depth requirements, scope/budget, Mermaid and citation rules, and validation checklist exactly. The topic (and optional specific file paths) to document:

$ARGUMENTS
