# Tech Stack

## This repository (the plugin)

deep-wiki is an **Agent Skills plugin**, not a conventional build project. Its source artifacts are Markdown and JSON:

- `skills/*/SKILL.md` (+ `references/*.md`) — auto-invoked skills; the portable source of truth.
- `commands/*.md` — slash-command entrypoints (`/deep-wiki:*`) that delegate to skills.
- `agents/*.md` — custom agents (`wiki-architect`, `wiki-writer`, `wiki-researcher`).
- `.claude-plugin/plugin.json` — plugin manifest (name, version, author).

There is no compiler, no test runner, and no dependency manifest at the repo root. "Building" the plugin means authoring and validating the Markdown prompt architecture. Distribution is via the Agent Skills marketplace (`/plugin install deep-wiki@skills`).

## Generated-site stack (the output, not this repo)

The packaging commands scaffold a separate Node project under `wiki/`:

Packaging is **multi-tool** behind the `wiki-site-core` skill — a generator-neutral core (`references/core-packaging.md`) plus a declarative adapter manifest per tool (`references/adapter-contract.md`), read identically by `build`, `deploy`, and CI. v1 adapters:

- **VitePress 1.x** (reference adapter, full parity) on **Node 20** — `vitepress-plugin-mermaid`, `mermaid` 11.x, `medium-zoom`. Runtime Mermaid (three-layer dark fix + click-to-zoom SVG overlay + focus mode). Output `wiki/.vitepress/dist`. The authoritative build code stays in `skills/wiki-vitepress/references/vitepress-build.md` (unchanged).
- **Nextra v4** (baseline parity) on **Node 22** — React/Next, native Mermaid via `@theguild/remark-mermaid`, MDX, static export to `wiki/out` + `.nojekyll`. Dark theme via shared design tokens; zoom/focus declared absent.

`/deep-wiki:build --tool <name>` selects the generator (default VitePress). First follow-up adapter: **Starlight** (Astro, Mermaid via `astro-mermaid`). Further follow-ups: Docusaurus, Fumadocs, Docus.

## Tooling notes

- The `please` plugin's own scripts run under **bun** (`bun run …`).
- Git host: GitHub (`chatbot-pf/deep-wiki`). graphite (`gt`) is installed but the stacked-PR workflow is disabled.
