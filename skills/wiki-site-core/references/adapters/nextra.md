# Adapter: Nextra v4

The second v1 adapter — a maximally-different generator from VitePress (React/Next vs Vue, native build-time-ish Mermaid vs runtime, MDX vs markdown-it). Its purpose is to prove the contract is generator-neutral. It ships at **baseline parity**: dark theme via the design tokens, dark Mermaid (Nextra native), intact citations, and the sidebar + onboarding structure. Click-to-zoom and focus mode are declared absent (VitePress-tier, optional).

## Manifest

```yaml manifest
tool: nextra
install_cmd: "npm install"
build_cmd: "next build"
output_dir: "out"
node_version: "22"
mermaid_strategy: native
parser_profile: mdx
dark_mode: forced
base_path:
  kind: next-config
  target: "next.config.mjs"
  key: "basePath"
extra_files: [".nojekyll"]
capabilities:
  dark_mermaid: true
  citations: true
  sidebar_onboarding: true
  click_to_zoom: false
  focus_mode: false
```

## Scaffold delta

Everything tool-independent (output structure, tokens, landing page, citations, sidebar/onboarding spec, llms.txt, post-processing) comes from `core-packaging.md`. Nextra-specific scaffolding:

- **`package.json`** — `next`, `react`, `react-dom`, `nextra`, `nextra-theme-docs`, `@theguild/remark-mermaid`. Scripts: `"build": "next build"`. (Node 22+.)
- **`next.config.mjs`** — wrap with `nextra({ ... })`; register `@theguild/remark-mermaid` in `mdxOptions.remarkPlugins` (this is the `native` Mermaid path — fenced ` ```mermaid ` blocks render with no client JS from the core). For static export set `output: 'export'`, `images: { unoptimized: true }`. For a project site, set `basePath` and `assetPrefix` (the manifest's `base_path` drives this; `deploy` injects the repo name).
- **Theme + tokens** — import a `styles.css` that defines the design tokens (`--bg`, `--surface`, `--card`, `--border`, `--accent`, `--text`, `--muted`) as CSS variables and maps them onto Nextra's theme CSS variables, forcing dark (`dark_mode: forced`). Fonts: Inter + JetBrains Mono. No hex literals are re-introduced — the tokens are the single source.
- **Mermaid (native)** — handled entirely by `@theguild/remark-mermaid`; dark mode follows the forced-dark theme. The core emits **no** Mermaid JS and skips the `mermaid_strategy: runtime` inline-style transform for this adapter.
- **Navigation** — render the core's sidebar + onboarding-first spec into Nextra `_meta` files (one per directory): the onboarding group first, then the numbered section folders in order. This backs the `sidebar_onboarding` baseline capability.
- **Pages / `parser_profile: mdx`** — the core emits the wiki pages as `.md` (not `.mdx`) and escapes bare `<T>` generics, so the generated Mermaid-heavy, citation-dense Markdown is consumed by Nextra without JSX-parser breakage. This resolves the T004 STOP condition: generic handling lives in the core's `mdx` profile, not in per-page hand-fixes.
- **Citations** — plain Markdown links render natively in Nextra; nothing adapter-specific is required. Baseline `citations: true`.
- **Static export extras** — emit `.nojekyll` (per `extra_files`) so GitHub Pages serves the Next `out/` directory's `_next/` assets.

## Declared gaps (baseline parity)

- **`click_to_zoom: false`** — the VitePress SVG-overlay zoom is not ported in v1. Mermaid still renders dark; diagrams are just not click-to-zoom.
- **`focus_mode: false`** — the reading focus-mode toggle is not ported in v1.

These are honest declarations, not failures (per the contract's capability tiers). The build succeeds and reports the gaps; a follow-up may add a framework-agnostic overlay.

## Verification (v1: manual smoke build)

On a fixture wiki (one dark Mermaid diagram, a `<T>` generic, a citation, an onboarding page):

- `next build` produces a non-empty `out/` static site.
- Mermaid renders in dark mode via the native remark plugin.
- The `<T>` generic page builds without MDX/JSX errors (core `mdx` profile handled it).
- Citation links resolve; the onboarding-first `_meta` nav is present.
- The build does not fail for the absent zoom/focus capabilities.
