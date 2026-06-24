# Adapter: VitePress (reference)

The reference adapter — VitePress expressed against the contract like any other tool, with no privileged path through the core. It declares the manifest below and owns the VitePress-specific delta; everything else comes from `core-packaging.md`.

**Regression bar (FR-007):** the VitePress build implementation is unchanged. The full, authoritative build code (`config.mts`, `theme/index.ts`, `custom.css`, `package.json`, the three-layer Mermaid dark fix, the click-to-zoom SVG overlay, focus mode) lives in [`skills/wiki-vitepress/references/vitepress-build.md`](../../../../wiki-vitepress/references/vitepress-build.md) and is consumed as-is. This adapter is a manifest + classification layer over that code, so `/deep-wiki:build` with no `--tool` produces a byte-for-byte equivalent site.

## Manifest

```yaml manifest
tool: vitepress
install_cmd: "npm install"
build_cmd: "npm run build"
output_dir: ".vitepress/dist"
node_version: "20"
mermaid_strategy: runtime
parser_profile: markdown-it
dark_mode: forced
base_path:
  kind: config-edit
  target: ".vitepress/config.mts"
  key: "base"
extra_files: []
capabilities:
  dark_mermaid: true
  citations: true
  sidebar_onboarding: true
  click_to_zoom: true
  focus_mode: true
```

## Core vs Adapter split

What `core-packaging.md` now owns (not duplicated here): the common `wiki/` output structure, the design tokens (palette + fonts), the developer-focused `index.md` landing page, plain-Markdown citations, the sidebar + onboarding-first spec, `llms.txt`/`llms-full.txt`/`AGENTS.md` emission, and the profile-keyed post-processing catalog. For `parser_profile: markdown-it` the core runs the `<br/>`→`<br>` and bare-`<T>` transforms; for `mermaid_strategy: runtime` it runs the Mermaid inline-style light→dark replacement.

What this adapter owns (the VitePress-specific delta — full code in `vitepress-build.md`):

- **`.vitepress/config.mts`** — `withMermaid()` wrapper, `appearance: 'dark'`, `outline`, `markdown.lineNumbers`, the Mermaid `themeVariables` (mapped from the design tokens), and the dynamic sidebar rendered from the core's sidebar spec into VitePress `sidebar` config.
- **`.vitepress/theme/index.ts`** — the click-to-zoom SVG overlay (poll async Mermaid SVGs, clone, fix `viewBox`, pan/zoom, listener cleanup) and the focus-mode toggle. These back the `click_to_zoom` and `focus_mode` capabilities.
- **`.vitepress/theme/custom.css`** — the dark theme CSS, the Mermaid CSS `!important` overrides (the second layer of the three-layer dark fix), and the zoom/focus CSS — all using the design-token values.
- **`package.json`** — `vitepress`, `vitepress-plugin-mermaid`, `mermaid`, `medium-zoom`.
- **Base path** — `deploy` sets `base` in `.vitepress/config.mts` per the manifest's `base_path` (`kind: config-edit`).

The three-layer Mermaid dark fix maps onto the split: layer 1 (`themeVariables`) and layer 2 (CSS `!important` overrides) are adapter delta; layer 3 (inline-style post-processing) is the core's `mermaid_strategy: runtime` transform.

## Verification

- Behavior-identity (FR-007) holds by construction: the build implementation in `vitepress-build.md` is not modified by this track; this adapter only adds the manifest + classification. Manual confirmation is a before/after smoke build of a fixture wiki (deferred per the plan's v1 verification approach).
- All capabilities declared `true` — VitePress is the full-parity reference.
