# Adapter: Astro Starlight

The third v1 adapter — Starlight, the documentation theme built on Astro. Like Nextra it proves the contract is generator-neutral (Astro islands + content collections vs Vue/VitePress and React/Next), and it ships at **baseline parity**: a light/dark theme that follows the OS preference by default (dark scheme recolored via the design tokens), dark Mermaid (Astro native integration), intact citations, and the sidebar + onboarding structure. Click-to-zoom and focus mode are declared absent (VitePress-tier, optional).

## Manifest

```yaml manifest
tool: starlight
install_cmd: "npm install"
build_cmd: "astro build"
output_dir: "dist"
node_version: "22"
mermaid_strategy: native
parser_profile: mdx
dark_mode: system
base_path:
  kind: config-edit
  target: "astro.config.mjs"
  key: "base"
extra_files: [".nojekyll"]
capabilities:
  dark_mermaid: true
  citations: true
  sidebar_onboarding: true
  click_to_zoom: false
  focus_mode: false
```

## Scaffold delta

Everything tool-independent (output structure, tokens, landing page, citations, sidebar/onboarding spec, llms.txt, post-processing) comes from `core-packaging.md`. Starlight-specific scaffolding:

- **`package.json`** — `astro`, `@astrojs/starlight`, and the Mermaid integration `astro-mermaid` (+ `mermaid`). Scripts: `"build": "astro build"`. (Node 22+ per current Astro; CI pins 22.)
- **`astro.config.mjs`** — register `@astrojs/starlight` and `astro-mermaid` in `integrations`. Set `site` and, for a project site, `base` (the manifest's `base_path` drives this; `deploy` injects the repo name into the `base` key). Configure the Starlight `sidebar` from the core's sidebar + onboarding-first spec, and point Starlight's `customCss` at the token stylesheet below.
- **Content location / `parser_profile: mdx`** — Starlight reads pages from the `docs` content collection at `src/content/docs/`. The core emits the wiki pages as `.md` (not `.mdx`) and escapes bare `<T>` generics, so the Mermaid-heavy, citation-dense Markdown is consumed by Astro's content pipeline without the JSX/HTML-tag parser dropping `<T>`. Each page needs Starlight frontmatter (`title`); the core's page metadata supplies it. The landing page maps to a `src/content/docs/index.md` (optionally with Starlight's `template: splash` hero).
- **Theme + tokens (default)** — by default the adapter uses Starlight's **stock theme recolored to the design tokens**: a `styles.css` defines the tokens (`--bg`, `--surface`, `--card`, `--border`, `--accent`, `--text`, `--muted`) as CSS variables and maps them onto Starlight's theming custom properties (`--sl-color-bg`, `--sl-color-bg-nav`, `--sl-color-text`, `--sl-color-accent`, `--sl-color-gray-*`, …). Fonts: Inter + JetBrains Mono. No hex literals are re-introduced for the dark scheme — the tokens are the single source. This default keeps the Starlight dark scheme visually consistent with the VitePress and Nextra adapters.
- **Theme mode (`dark_mode: system`)** — Starlight ships a light/dark/**auto** theme switcher; the adapter **keeps it enabled** and leaves the default at **auto, which follows the OS `prefers-color-scheme`** (Starlight's out-of-the-box behavior — do not force `data-theme` or suppress `ThemeSelect`). The token stylesheet recolors the dark scheme; Starlight's native light scheme backs light mode. Users can still override via the switcher.
- **Theme selection (`--theme <plugin>`)** — Starlight has an ecosystem of community themes (see <https://starlight.astro.build/resources/themes/>), each shipped as a Starlight **plugin**. The adapter accepts an optional `--theme <plugin>` argument (resolved by `build` and passed to this adapter; meaningful only for `--tool starlight`):
  - **omitted (default):** stock theme + token recolor above. Cross-adapter consistency; no extra dependency.
  - **a known community theme:** add its package to `package.json`, register it in Starlight's `plugins: [...]` array in `astro.config.mjs`, and let the theme own the palette. Examples (confirm the exact package name + install snippet on the themes page): `starlight-theme-rapide` (VS Code Vitesse), `starlight-theme-black` / `starlight-theme-nova` (shadcn/ui-style), `starlight-theme-flexoki`, `starlight-theme-nord`. The deep-wiki token stylesheet is then reduced to **accent + font overrides only** (or dropped) so it does not fight the theme's colors; the OS-following (auto/`system`) mode still applies. If `--theme` names a plugin not recognized here, install the named package and register it, but report that token mapping is the theme's responsibility (the deep-wiki palette is not layered on an unknown theme).
- **Mermaid (native)** — handled by the `astro-mermaid` integration, which renders fenced ` ```mermaid ` blocks and follows the **active** theme (OS preference by default, re-rendering when the user switches scheme; set its `theme` from the design tokens and enable `autoTheme` so it tracks the active light/dark scheme). The core emits **no** Mermaid JS and skips the `mermaid_strategy: runtime` inline-style transform for this adapter. _(Build-time pre-render via `rehype-mermaid` is an alternative once the contract's `build-time` strategy lands; v1 uses the `native` integration.)_
- **Navigation** — render the core's sidebar + onboarding-first spec into the Starlight `sidebar` config array (`astro.config.mjs`): the onboarding group first, then the numbered section folders in order. This backs the `sidebar_onboarding` baseline capability. (Starlight can also autogenerate from directory order, but the explicit spec preserves onboarding-first ordering.)
- **Citations** — plain Markdown links render natively in Starlight; nothing adapter-specific is required. Baseline `citations: true`.
- **Static export extras** — Astro's default static build emits to `dist/` with assets under `_astro/`; emit `.nojekyll` (per `extra_files`) so GitHub Pages serves the underscore-prefixed asset directory.

## Declared gaps (baseline parity)

- **`click_to_zoom: false`** — the VitePress SVG-overlay zoom is not ported in v1. Mermaid still renders dark; diagrams are just not click-to-zoom.
- **`focus_mode: false`** — the reading focus-mode toggle is not ported in v1.

These are honest declarations, not failures (per the contract's capability tiers). The build succeeds and reports the gaps; a follow-up may add a framework-agnostic overlay.

## Verification (v1: manual smoke build)

On a fixture wiki (one dark Mermaid diagram, a `<T>` generic, a citation, an onboarding page):

- `astro build` produces a non-empty `dist/` static site.
- Mermaid renders in dark mode via the `astro-mermaid` integration.
- The `<T>` generic page builds without the content parser dropping the tag (core `mdx` profile handled it).
- Citation links resolve; the onboarding-first Starlight `sidebar` is present; the theme follows the OS preference (auto) with the light/dark switcher available.
- The build does not fail for the absent zoom/focus capabilities.
