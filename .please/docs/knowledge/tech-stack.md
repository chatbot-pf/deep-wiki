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

- **VitePress 1.x** (current sole packager) on **Node 20** — `vitepress-plugin-mermaid`, `mermaid` 11.x, `medium-zoom`. Output `wiki/.vitepress/dist`. Deployed to GitHub Pages via a generated Actions workflow.
- **Mermaid** for diagrams, with a dark-mode "three-layer fix" and a custom click-to-zoom SVG overlay.

Planned additional packagers (behind a generator-neutral core + adapter contract): **Nextra v4** (React/Next, native Mermaid), **Starlight** (Astro, Mermaid via `astro-mermaid`), and others (Docusaurus, Fumadocs, Docus).

## Tooling notes

- The `please` plugin's own scripts run under **bun** (`bun run …`).
- Git host: GitHub (`chatbot-pf/deep-wiki`). graphite (`gt`) is installed but the stacked-PR workflow is disabled.
