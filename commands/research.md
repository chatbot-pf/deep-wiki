---
description: Conduct multi-turn deep research on a specific topic — traces actual code paths with zero tolerance for shallow analysis
argument-hint: [topic]
disable-model-invocation: true
---

# Deep Wiki: Deep Research

> **Note (Claude Code only):** This command is a thin entrypoint. The full procedure lives in the `wiki-researcher` skill, which is the single source of truth and the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Invoke the **`wiki-researcher`** skill to conduct a comprehensive, multi-turn investigation of the topic below — follow its source-repository resolution, core invariants, 5-iteration research cycle, running knowledge map, and evidence standard exactly. The research topic:

$ARGUMENTS
