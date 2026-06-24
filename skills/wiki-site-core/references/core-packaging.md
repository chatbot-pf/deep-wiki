# Core Packaging — Tool-Independent Logic

Everything the wiki packager does that does **not** depend on the chosen documentation generator. Written once here; every adapter inherits it. Tool-specific scaffolding, config, and theming live in the per-adapter files under `adapters/`.

A change to anything in this file lands once and applies to every adapter — that is the point of the core (FR-002).

## Common Output Structure

Every adapter scaffolds a `wiki/` directory. These entries are tool-independent; the generator-specific config directory (`.vitepress/`, `.next/` config, `astro.config`, etc.) is added by the adapter.

```
wiki/
├── index.md                 # developer-focused landing page (see below)
├── llms.txt                 # LLM-friendly links + descriptions
├── llms-full.txt            # LLM-friendly full inlined content
├── AGENTS.md                # agent instructions for the wiki folder
├── CLAUDE.md                # companion pointer to AGENTS.md
├── onboarding/              # audience-tailored onboarding guides
│   ├── index.md
│   ├── contributor-guide.md
│   ├── staff-engineer-guide.md
│   ├── executive-guide.md
│   └── product-manager-guide.md
├── {NN}-{section-name}/     # numbered section folders with generated pages
│   └── {page-name}.md
└── <adapter config dir>     # added by the adapter (e.g. .vitepress/)
```

## Design Tokens (canonical — adapters map these)

The dark palette and fonts are the project's visual identity, defined **once** as tokens. Each adapter maps these tokens into its own theming system (VitePress `themeVariables` + CSS vars, Nextra CSS vars, etc.). Adapters reference the token names; they do not re-hardcode hex literals — a palette change here re-skins every site.

| Token | Value | Role |
|-------|-------|------|
| `--bg` | `#0d1117` | Page background |
| `--surface` | `#161b22` | Elevated surface |
| `--surface-2` | `#1c2333` | Secondary surface |
| `--card` | `#2d333b` | Card / Mermaid node |
| `--border` | `#30363d` | Borders |
| `--accent` | `#6d5dfc` | Brand accent |
| `--text` | `#e6edf3` | Primary text |
| `--muted` | `#8b949e` | Muted text / lines / arrows |
| `--font-base` | `Inter` | Body font |
| `--font-mono` | `JetBrains Mono` | Monospace font |

## Landing Page (index.md) — developer-focused, NOT marketing

`index.md` MUST be a technical wiki home page: no `hero:` frontmatter, no taglines, no call-to-action buttons, no marketing copy. Include: YAML frontmatter (`title`, `description`); a 1–2 sentence technical description; a Quick Start with actual runnable commands; an architecture Mermaid diagram with `<!-- Sources: ... -->`; a Documentation Map table linking the sections; a Key Files table with source citations; a Tech Stack table. This content is identical across adapters; only the Mermaid render path differs.

## Source-Linked Citations

Generated pages carry `file:line` citations: remote repos as `[file:line](REPO_URL/blob/BRANCH/file#Lline)`, local as `(file:line)`. Citations are emitted as **plain Markdown links**, which are tool-neutral and render under every adapter's `parser_profile`. The core preserves them through post-processing; an adapter must not strip or rewrite them. `citations` is a baseline capability — every adapter renders them intact.

## Dynamic Sidebar + Onboarding-First Ordering

The sidebar is generated from the section structure, as a tool-independent **spec** each adapter renders in its own config format:

- ONBOARDING section is always first and uncollapsed, with the four audience guides (Contributor, Staff Engineer, Executive, Product Manager).
- Then numbered sections (`01-getting-started`, `02-architecture`, …), each a collapsible group; first 3–4 uncollapsed, rest collapsed.

`sidebar_onboarding` is a baseline capability. The adapter translates this spec into VitePress `sidebar`, Nextra `meta.json`, etc.

## llms.txt / AGENTS.md Emission

The core emits, tool-independently:

- `llms.txt` — links + descriptions for every wiki page (also served at `/llms.txt` by adapters whose static server exposes a `public/` dir).
- `llms-full.txt` — full inlined content for comprehensive agent context.
- `AGENTS.md` (+ `CLAUDE.md` pointer) — agent instructions referencing the wiki docs.

These are placed at standard discoverable paths and are identical across adapters.

## Post-Processing — Profile-Keyed Transform Catalog

Before building, the core fixes generated Markdown. Each transform is tagged with the `parser_profile` or `mermaid_strategy` that triggers it (per the adapter's manifest), so the core owns the catalog and the adapter's manifest selects which transforms run. This is the STOP-condition resolution from T002: a transform that depends on a tool specific is gated by profile/strategy, **not** duplicated into each adapter and **not** hardcoded to VitePress.

| Transform | Triggered when | What it does |
|-----------|----------------|--------------|
| Validate Mermaid hex colors | all profiles | Ensure every Mermaid hex is 3 or 6 digits. |
| Ensure frontmatter | all profiles | Every page has YAML frontmatter with `title` + `description`. |
| `<br/>` → `<br>` | `parser_profile: markdown-it` | Self-closing tags break Vue's template compiler. |
| Wrap/escape bare `<T>` generics | `parser_profile: markdown-it` or `mdx` | A bare `Array<T>` in prose is read as a tag (Vue) or JSX element (MDX); backtick-wrap it. For `mdx`, also emit `.md` (not `.mdx`) so the local-md path skips JSX parsing. |
| Mermaid inline-style light→dark replacement | `mermaid_strategy: runtime` | Replace light-mode `style` directives in Mermaid blocks with dark equivalents (`#e1f5ff`→`#1a3a4a`, …, append `,color:#e6edf3`). Only needed when Mermaid renders client-side; `native`/`build-time` strategies skip it. |

The Mermaid dark-mode handling beyond inline styles (theme variables, CSS `!important` overrides, the async-poll zoom overlay) is **not** core — it is the `runtime` strategy's adapter delta (see `adapters/vitepress.md`). The core only owns the profile-gated text transforms above.

## What the Core Does NOT Own

These are adapter responsibilities (declared via the manifest, implemented in the adapter delta): the generator config file(s), the theme/scaffold, the Mermaid render wiring for the declared `mermaid_strategy`, the `build_cmd`/`output_dir`/`node_version`/`base_path`/`extra_files`, and the VitePress-tier interactions (click-to-zoom, focus mode) for adapters that declare them.
