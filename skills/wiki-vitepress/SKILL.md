---
name: wiki-vitepress
description: Packages generated wiki Markdown into a VitePress static site with dark theme, dark-mode Mermaid diagrams with click-to-zoom, and production build output. The VitePress adapter of the wiki-site-core multi-tool packager. Use when the user wants to create a browsable VitePress website from generated wiki pages.
license: MIT
user-invocable: false
metadata:
  author: Microsoft
  version: "1.1.0"
---

# Wiki VitePress Packager (VitePress adapter)

Packaging is now multi-tool. The generator-neutral logic and the per-tool adapter contract live in the **`wiki-site-core`** skill; VitePress is its **reference adapter**. This skill is the VitePress entry point — it keeps the existing trigger ("build a site") and points at the core plus the VitePress adapter.

## When to Activate

- User asks to "build a site" or "package as VitePress".
- User runs `/deep-wiki:build` (with no `--tool`, or `--tool vitepress`).
- User wants a browsable VitePress HTML output from generated wiki pages.

For other generators (e.g. `--tool nextra`), `/deep-wiki:build` loads `wiki-site-core` + the requested adapter directly.

## Where the procedure lives

- **Core (tool-independent):** [`skills/wiki-site-core/SKILL.md`](../wiki-site-core/SKILL.md) → `references/core-packaging.md` — post-processing, citations, design tokens, sidebar + onboarding, `llms.txt`/`AGENTS.md`, landing page.
- **Adapter contract:** [`skills/wiki-site-core/references/adapter-contract.md`](../wiki-site-core/references/adapter-contract.md) — the capability manifest read by build/deploy/CI.
- **VitePress adapter:** [`skills/wiki-site-core/references/adapters/vitepress.md`](../wiki-site-core/references/adapters/vitepress.md) — the VitePress manifest + the core/adapter delta classification.
- **Full VitePress build code (authoritative, unchanged):** [`references/vitepress-build.md`](references/vitepress-build.md) — the complete `config.mts`, `theme/index.ts` (image zoom, Mermaid SVG zoom overlay, focus mode), `custom.css`, `package.json`, post-processing, and the wiki landing page. Load it when actually scaffolding or building the VitePress site.

## VitePress essentials (overview)

VitePress renders a dark-theme static site with dark-mode Mermaid diagrams (three-layer fix: `themeVariables` + CSS `!important` overrides + inline-style post-processing), a click-to-zoom SVG overlay, and a focus-mode toggle. These are the VitePress adapter's full-parity capabilities. The canonical dark palette and the three-layer Mermaid fix are detailed in `references/vitepress-build.md`; under the multi-tool model the palette is sourced from the core's design tokens and the inline-style post-processing is the core's `mermaid_strategy: runtime` transform.

## Build

```bash
cd wiki && npm install && npm run build
```

Output goes to `wiki/.vitepress/dist/`. Preview with `npm run preview`. See `references/vitepress-build.md` for the complete, authoritative implementation.

## Known Gotchas

- Mermaid renders async — SVGs don't exist when `onMounted` fires. Must poll.
- `isCustomElement` compiler option for bare `<T>` causes worse crashes — do NOT use it.
- Node text in Mermaid uses inline `style` with highest specificity — CSS alone won't fix it.
- `enhanceApp()` runs during SSR where `document` doesn't exist — use `setup()` only.
