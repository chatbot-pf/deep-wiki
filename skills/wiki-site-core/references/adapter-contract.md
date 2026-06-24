# Adapter Contract — Manifest Schema & Consumption

The contract that makes wiki packaging multi-tool. Each documentation generator is expressed as one **adapter** = a declarative **capability manifest** + a tool-specific scaffold/theme delta. `build`, `deploy`, and CI read the manifest as the **single source of truth** for tool-specific values — none of them hardcode a tool's build command, output directory, or base-path logic.

## Adapter File Shape

Each adapter lives at `references/adapters/<tool>.md` and has two parts:

1. A fenced **`yaml manifest`** block at the head — the declarative fields below.
2. The tool-specific delta — scaffold files, theme/token mapping, Mermaid wiring — everything the core does not own.

The manifest is Markdown-native (a fenced YAML block an agent reads); no runtime parser is introduced. The plugin's "source" is prompts/Markdown, so the contract is consumed by reading, the same way the rest of the skills are.

## Manifest Schema

```yaml manifest
tool: vitepress                 # adapter id (matches the --tool value and the file name)
install_cmd: "npm install"      # dependency install, run in the scaffolded wiki/ dir
build_cmd: "npm run build"      # static build command
output_dir: ".vitepress/dist"   # build output, relative to wiki/
node_version: "20"              # required Node major version
mermaid_strategy: runtime       # runtime | native | build-time  (see "Mermaid Strategies")
parser_profile: markdown-it     # markdown-it | mdx | mdc  (see "Parser Profiles")
dark_mode: forced               # forced | toggle | system  — how the adapter applies the dark palette
base_path:                      # how the deployed base path is set (see "Base-Path Mechanisms")
  kind: config-edit             # config-edit | next-config | env
  target: ".vitepress/config.mts"
  key: "base"
extra_files: []                 # files the build/deploy must emit beyond the site (e.g. [".nojekyll"])
capabilities:                   # which signature features this adapter provides (see "Capability Tiers")
  dark_mermaid: true            # baseline (required)
  citations: true               # baseline (required)
  sidebar_onboarding: true      # baseline (required)
  click_to_zoom: true           # VitePress-tier (optional)
  focus_mode: true              # VitePress-tier (optional)
```

### Field semantics

| Field | Meaning |
|-------|---------|
| `tool` | Adapter id; matches `--tool <id>` and the adapter file name. |
| `install_cmd` / `build_cmd` | Commands run in the scaffolded `wiki/` directory. `deploy` and CI read these — never assume `npm run build`. |
| `output_dir` | Build artifact directory relative to `wiki/`; `deploy` uploads this — never assume `.vitepress/dist`. |
| `node_version` | Required Node major; `deploy`/CI pin the runtime from here. |
| `mermaid_strategy` | How Mermaid is rendered; selects which core/adapter Mermaid path runs (below). |
| `parser_profile` | The Markdown dialect the generator consumes; selects which core post-processing transforms apply (below). |
| `dark_mode` | How the adapter applies the canonical dark palette tokens. |
| `base_path` | Declarative description of the deployed base-path mechanism (below) — replaces hardcoded VitePress `base` editing. |
| `extra_files` | Files the build/deploy must additionally emit (e.g. `.nojekyll` for Next/Astro static export on Pages). |
| `capabilities` | Declares which signature features the adapter provides; baseline features are required, VitePress-tier are optional (below). |

## Mermaid Strategies

`mermaid_strategy` declares how an adapter renders dark-mode Mermaid:

- `runtime` — the generator renders Mermaid client-side as live SVG; the core's runtime-overlay path (three-layer dark fix + async-poll zoom) applies. **VitePress.**
- `native` — the generator has built-in Mermaid (e.g. a remark plugin) that handles dark mode; the core emits no Mermaid JS. **Nextra (`@theguild/remark-mermaid`).**
- `build-time` — Mermaid is pre-rendered to (dual light/dark) SVG at build time; the core emits no client Mermaid JS. _(Reserved for follow-up adapters; not implemented in v1.)_

v1 implements `runtime` and `native` only. A new value is added when an adapter needs it.

## Parser Profiles

`parser_profile` selects which core post-processing transforms apply to the generated Markdown:

- `markdown-it` — VitePress. The core applies the `<br/>`→`<br>` fix and wraps bare `<T>` generics.
- `mdx` — Nextra / React generators. The core escapes bare `<T>` generics that the JSX parser would treat as elements, and emits `.md` (not `.mdx`) so the local-md path avoids JSX parsing entirely.
- `mdc` — Docus / Nuxt Content. The core guards the `---` props/slot delimiter. _(Reserved; not implemented in v1.)_

v1 implements `markdown-it` and `mdx`. See `core-packaging.md` for the transforms each profile triggers.

## Base-Path Mechanisms

`base_path.kind` makes the deployed base path declarative rather than VitePress-specific:

- `config-edit` — set `key` in the config file at `target` (VitePress: `base` in `.vitepress/config.mts`).
- `next-config` — set `basePath` + `assetPrefix` in `next.config` (Nextra / Next static export).
- `env` — set an environment variable named by `key` at build time (Docus: `NUXT_APP_BASE_URL`). _(Reserved.)_

`deploy` reads `base_path` to decide how to inject the project-site base path; it never edits `config.mts` unconditionally.

## Capability Tiers

Not every adapter must reproduce every signature feature. Capabilities split into two tiers:

- **Baseline (required of every adapter):** `dark_mermaid`, `citations`, `sidebar_onboarding`, plus the dark palette via tokens. An adapter that cannot reach baseline is not "supported".
- **VitePress-tier (optional):** `click_to_zoom`, `focus_mode`. An adapter declares these `true`/`false`; declaring `false` is honest, not a failure — the build still succeeds, and the missing capability is reported, not silently dropped.

The VitePress adapter declares all capabilities `true`. The Nextra adapter declares baseline `true` and the VitePress-tier capabilities `false` (v1 baseline parity).

## Consumption Rules

- **`build`** resolves the target tool (order: `--tool` arg → config default → `vitepress`), loads the core + chosen adapter, runs the core post-processing for the adapter's `parser_profile`, scaffolds via the adapter, and runs `install_cmd` then `build_cmd`.
- **`deploy`** generates the CI workflow from the manifest: `build_cmd`, `output_dir` (artifact path), `node_version`, `base_path`, and `extra_files`. No VitePress literal appears in the generated workflow unless VitePress is the chosen tool.
- **CI / parity checks** read the same manifest, so build, deploy, and CI never disagree about a tool's values.

**Invariant:** no tool-specific build command, output directory, or base-path value lives anywhere outside an adapter manifest.

## Adding an Adapter

1. Create `references/adapters/<tool>.md`.
2. Fill the `yaml manifest` block (every field above).
3. Write the tool-specific delta: scaffold files, map the dark palette **tokens** into the generator's theming, wire Mermaid per the declared `mermaid_strategy`, and ensure citations render.
4. Declare `capabilities` honestly — baseline must be `true`; VitePress-tier may be `false`.
5. Verify with a smoke build of a fixture wiki (one dark Mermaid diagram, a `<T>` generic, a citation, an onboarding page): the site builds, Mermaid renders dark, citations resolve, the sidebar/onboarding structure is present.

No edits to `core-packaging.md`, `build.md`, or `deploy.md` are needed — they consume the manifest generically.
