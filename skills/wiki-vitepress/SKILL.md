---
name: wiki-vitepress
description: Packages generated wiki Markdown into a VitePress static site with dark theme, dark-mode Mermaid diagrams with click-to-zoom, and production build output. Use when the user wants to create a browsable website from generated wiki pages.
license: MIT
user-invocable: false
metadata:
  author: Microsoft
  version: "1.0.0"
---

# Wiki VitePress Packager

Transform generated wiki Markdown files into a polished VitePress static site with a dark theme and interactive Mermaid diagrams.

## When to Activate

- User asks to "build a site" or "package as VitePress"
- User runs the `/deep-wiki:build` command
- User wants a browsable HTML output from generated wiki pages

## Full Implementation Reference

The complete, authoritative build spec — VitePress `config.mts`, `theme/index.ts` (image zoom, Mermaid SVG zoom overlay, focus mode), full `custom.css`, `package.json`, post-processing, and the wiki landing page — lives in **[references/vitepress-build.md](references/vitepress-build.md)**. Load and follow it when actually scaffolding or building the site. This page is the overview; the reference is the source of truth for all code.

## Overview & Key Principles

### Output Structure

Scaffold a `wiki/` directory containing `package.json`, `.gitignore`, `index.md` (developer-focused landing page — **not** a marketing hero page), the generated section folders, `onboarding/`, and `.vitepress/` (`config.mts` + `theme/index.ts` + `theme/custom.css`). See the reference for the full tree.

### Dark-Mode Palette (canonical — use everywhere)

| Element | Background | Border | Text |
|---------|-----------|--------|------|
| Page background | `#0d1117` | — | `#e6edf3` |
| Elevated surface | `#161b22` | `#30363d` | `#e6edf3` |
| Card / Mermaid node | `#2d333b` | `#6d5dfc` | `#e6edf3` |
| Secondary surface | `#1c2333` | `#6d5dfc` | `#e6edf3` |
| Lines / arrows | — | `#8b949e` | — |
| Brand accent | — | `#6d5dfc` | — |

### Dark-Mode Mermaid: Three-Layer Fix

Mermaid theming needs all three layers (full code in the reference):

1. **Theme variables** — set `mermaid.theme: 'dark'` + comprehensive `themeVariables` in `config.mts`
2. **CSS overrides** — force dark fills on `.mermaid` SVG shapes with `!important` in `custom.css`
3. **Inline-style post-processing** — replace light-mode `style` directives in Mermaid blocks before build

### Click-to-Zoom

Mermaid renders `<svg>` (not `<img>`), so `medium-zoom` won't work — a custom fullscreen overlay is required. Implementation notes that matter:

- Use `setup()` + `onMounted` + route watcher, **not** `enhanceApp()` (no DOM during SSR)
- **Poll** for async-rendered Mermaid SVGs (retry up to 20 × 500ms)
- **Clone** the SVG (don't move it) and fix a missing `viewBox` from `getBBox()`
- Mark containers with `data-zoom-attached` to avoid duplicate handlers on route changes
- **Remove every `document`-level listener (`keydown`, `mousemove`, `mouseup`) in the close handler** so listeners don't leak across opens

### Post-Processing Rules

Before building, scan all `.md` files and fix:
- Replace `<br/>` with `<br>` (Vue template compiler compatibility)
- Wrap bare `<T>` generic parameters in backticks outside code fences
- Validate Mermaid hex colors (3 or 6 digits)
- Ensure every page has YAML frontmatter with `title` and `description`

### Build

```bash
cd wiki && npm install && npm run build
```

Output goes to `wiki/.vitepress/dist/`. For preview: `npm run preview`.

## Known Gotchas

- Mermaid renders async — SVGs don't exist when `onMounted` fires. Must poll.
- `isCustomElement` compiler option for bare `<T>` causes worse crashes — do NOT use it
- Node text in Mermaid uses inline `style` with highest specificity — CSS alone won't fix it
- `enhanceApp()` runs during SSR where `document` doesn't exist — use `setup()` only
