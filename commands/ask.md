---
description: Ask a question about the repository using wiki context and source file references
argument-hint: [question]
disable-model-invocation: true
---

# Deep Wiki: Repository Q&A

> **Note (Claude Code only):** This command is a thin entrypoint. The full procedure lives in the `wiki-qa` skill, which is the single source of truth and the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Invoke the **`wiki-qa`** skill to answer the following question about this repository, following its source-repository resolution, evidence rules, and response format exactly:

$ARGUMENTS
