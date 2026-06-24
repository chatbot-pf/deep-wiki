---
description: Generate a Node.js build script that converts the VitePress wiki to Azure DevOps Wiki-compatible markdown in dist/ado-wiki/. Transforms Mermaid syntax, strips front matter, fixes links.
disable-model-invocation: true
---

# Deep Wiki: Azure DevOps Wiki Export

> **Note (Claude Code only):** This command is a thin entrypoint. The full procedure lives in the `wiki-ado-convert` skill, which is the single source of truth and the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Invoke the **`wiki-ado-convert`** skill to generate a zero-dependency Node.js build script that converts the VitePress wiki into Azure DevOps Wiki-compatible markdown under `dist/ado-wiki/` — follow its source-repository resolution, pre-flight scan, transformation functions, index-page generation, `.order` files, citation preservation, and verification checklist exactly. **Source files are never modified.**

$ARGUMENTS
