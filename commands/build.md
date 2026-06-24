---
description: Package generated wiki pages into a static documentation site (VitePress, Nextra, …) with a dark theme, dark-mode Mermaid diagrams, and source citations
disable-model-invocation: true
---

# Deep Wiki: Build Site

> **Note (Claude Code only):** This command is a thin entrypoint. The procedure lives in the **`wiki-site-core`** skill (generator-neutral core + adapter contract) and its references — the portable unit that also works in GitHub Copilot CLI, Codex, Antigravity, and other Agent Skills runtimes.

Package the generated wiki Markdown into a static documentation site through the **`wiki-site-core`** skill: a generator-neutral core plus a per-tool adapter.

## Tool selection

Resolve the target generator in this order, then load `wiki-site-core` + the chosen adapter:

1. A `--tool <name>` argument, if present (e.g. `--tool nextra`).
2. Otherwise the **default: `vitepress`** — running with no argument produces the same dark-theme VitePress site as before (the regression-safe default).

Available adapters: `vitepress` (reference — full parity, click-to-zoom + focus mode) and `nextra` (baseline parity — dark theme, native dark Mermaid, citations, sidebar/onboarding; no zoom/focus). If `--tool` names an unknown adapter, stop and list the available ones.

## Build

1. Load `wiki-site-core` (`SKILL.md` → `references/adapter-contract.md`, `references/core-packaging.md`).
2. Load the chosen adapter at `references/adapters/<tool>.md` and read its manifest (`install_cmd`, `build_cmd`, `output_dir`, `parser_profile`, `mermaid_strategy`, etc.).
3. Run the core packaging: post-processing for the adapter's `parser_profile` and `mermaid_strategy`, citations, design tokens, sidebar + onboarding spec, landing page, `llms.txt`/`AGENTS.md`.
4. Scaffold via the adapter's delta, then run its `install_cmd` and `build_cmd`. Output goes to the adapter's `output_dir`.

For the VitePress adapter, the full authoritative build code lives in [`skills/wiki-vitepress/references/vitepress-build.md`](../skills/wiki-vitepress/references/vitepress-build.md) (unchanged) — load it for the complete config, theme, CSS, and scripts.

$ARGUMENTS
