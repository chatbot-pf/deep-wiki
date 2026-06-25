---
name: wiki-site-core
description: Packages generated wiki Markdown into a static documentation site through a generator-neutral core plus per-tool adapter manifests. Holds the tool-independent logic (post-processing, citations, dark palette tokens, sidebar/onboarding, llms.txt emission, landing page); each documentation generator (VitePress, Nextra, …) is a thin adapter. Use when building or packaging a wiki site, or adding support for a new documentation tool.
license: MIT
user-invocable: false
metadata:
  author: Minsu Lee
  version: "1.0.0"
---

# Wiki Site Core

Transform generated wiki Markdown into a polished static documentation site — through **one generator-neutral core** plus a **thin per-tool adapter** for each documentation generator. The core owns everything that does not depend on the chosen tool; an adapter owns only what differs.

This skill replaces the VitePress-only packaging path with a multi-tool contract. VitePress is now one adapter among others (the reference adapter), not a privileged substrate.

## When to Activate

- User asks to "build a site" or "package as a documentation site".
- User runs the `/deep-wiki:build` command (which resolves a target tool and loads the core + that adapter).
- User wants a browsable HTML output from generated wiki pages.
- User wants to add support for a new documentation generator (write a new adapter against the contract).

## How It Fits Together

```
commands/build.md   →  resolve --tool (default vitepress)  ┐
commands/deploy.md   →  read chosen adapter's manifest      ├─→  wiki-site-core  ──→  adapter(tool)
CI                   →  read chosen adapter's manifest      ┘     (tool-independent)     (tool-specific delta)
```

- **Core** (`references/core-packaging.md`) — Markdown post-processing, source-linked citations, the canonical dark palette + fonts as **design tokens**, dynamic sidebar with onboarding-first ordering, `llms.txt` / `llms-full.txt` / `AGENTS.md` emission, and the developer-focused landing page. Written once; applies to every adapter.
- **Adapter contract** (`references/adapter-contract.md`) — the declarative **capability manifest** every adapter declares, and how `build`, `deploy`, and CI consume it. This is the single source of truth for tool-specific values — no tool's build command, output directory, or base-path logic is hardcoded in the commands.
- **Adapters** (`references/adapters/<tool>.md`) — one per generator. Each carries a YAML manifest block at its head followed by the tool-specific scaffold/theme delta. v1 ships `vitepress` (reference), `nextra`, and `starlight` (Astro Starlight).

## Reference Map

| File | Purpose |
|------|---------|
| [`references/adapter-contract.md`](references/adapter-contract.md) | The manifest schema, consumption rules, parity tiers, and how to add an adapter |
| [`references/core-packaging.md`](references/core-packaging.md) | Tool-independent packaging logic (post-processing, citations, tokens, sidebar, llms.txt, landing page) |
| [`references/adapters/vitepress.md`](references/adapters/vitepress.md) | VitePress reference adapter — manifest + runtime-Mermaid / zoom / focus delta |
| [`references/adapters/nextra.md`](references/adapters/nextra.md) | Nextra v4 adapter — manifest + native-Mermaid / MDX delta (baseline parity) |
| [`references/adapters/starlight.md`](references/adapters/starlight.md) | Astro Starlight adapter — manifest + native-Mermaid (`astro-mermaid`) / content-collection delta (baseline parity) |

## Build

`/deep-wiki:build [--tool <name>]` resolves the target (default `vitepress`), loads the core plus the chosen adapter, runs the core post-processing, scaffolds via the adapter, and builds. See the adapter contract for the resolution order and per-tool build commands.

## Adding a Tool

Adding a documentation generator is writing one adapter file under `references/adapters/<tool>.md` — a manifest block plus a theme/scaffold delta — and declaring which capabilities it provides. No change to the core or to the command control flow. See `references/adapter-contract.md` → "Adding an Adapter".
