# Multi-tool wiki packaging: shared core + adapter contract

> Track: multi-tool-wiki-packaging-20260624
> Type: refactor
> Origin: docs/brainstorms/2026-06-24-multi-tool-wiki-packaging-requirements.md

## Overview

Refactor the deep-wiki site-packaging layer so it is no longer welded to VitePress. Extract the tool-independent packaging logic into a generator-neutral **core**, and express each documentation-site generator as a thin **adapter manifest** read identically by `commands/build.md`, `commands/deploy.md`, and CI. Re-express VitePress as the reference adapter (not a privileged substrate), and prove the abstraction by adding a second, maximally-different adapter (Nextra v4) at a baseline parity bar. The result turns "support N tools" from N forks of a ~650-line VitePress reference into one core plus N small descriptors.

## Scope

In scope for v1:

- A generator-neutral packaging core holding the tool-independent logic.
- A declarative adapter-manifest contract consumed uniformly by build, deploy, and CI.
- VitePress refactored into the reference adapter, with its current output preserved.
- A second adapter, Nextra v4, at baseline parity.
- An explicit tool selector for build and deploy, defaulting to VitePress.

## Functional Requirements

- **FR-001**: The packaging core MUST hold the tool-independent logic — Markdown post-processing, source-linked citations, the dark palette and fonts as design tokens, dynamic sidebar with onboarding-first ordering, `llms.txt` / `llms-full.txt` / `AGENTS.md` emission, and the developer-focused landing page. (R1)
- **FR-002**: A cross-cutting or palette/token change MUST be made once in the core and apply to every adapter; no adapter re-implements core logic. (R2)
- **FR-003**: Each supported tool MUST be expressed as one declarative adapter manifest capturing at least: install command, build command, output directory, Mermaid strategy, dark-mode handling, required Node version, base-path mechanism, content/parser profile, and extra emitted files (e.g. `.nojekyll`). (R3)
- **FR-004**: `build`, `deploy`, and CI MUST read the same manifest as the single source of truth for tool-specific values, and MUST NOT hardcode any tool's build command, output directory, or base-path logic. (R4)
- **FR-005**: The manifest MUST define `mermaid_strategy` and `parser_profile` as fields; v1 MUST implement only the values its two adapters use — Mermaid `runtime` (VitePress) and `native` (Nextra); parser `markdown-it` (VitePress) and `mdx` (Nextra). (R5)
- **FR-006**: VitePress MUST be re-expressed as the reference adapter against the contract, with no privileged path through the core. (R6)
- **FR-007**: After the refactor, the output of `/deep-wiki:build` with no tool selected MUST be behavior-identical to the pre-refactor VitePress output. (R7)
- **FR-008**: A Nextra v4 adapter MUST ship at baseline parity: it renders the wiki with the dark theme via the shared palette tokens, dark Mermaid via Nextra's native rendering, intact source citations, and the sidebar + onboarding structure. (R8)
- **FR-009**: Click-to-zoom and focus mode MUST be declared VitePress-tier capabilities, not required of every adapter; each adapter MUST declare which of them it provides. (R9)
- **FR-010**: `build` and `deploy` MUST resolve the target tool via an explicit selector (e.g. a `--tool` argument) defaulting to VitePress. (R10)
- **FR-011**: `deploy` MUST generate a workflow parameterized from the chosen adapter's manifest (build command, output directory, Node version, base-path mechanism, extra files), replacing the VitePress-hardcoded workflow. (R11)

## Acceptance Criteria (EARS)

1. **AC-001** — When a user runs `/deep-wiki:build` with no tool argument after the refactor, the system shall produce a site behavior-identical to the pre-refactor VitePress output (dark theme, dark Mermaid, click-to-zoom, focus mode, structure). (FR-007)
2. **AC-002** — When a user builds the same generated wiki with the Nextra adapter, the system shall render the shared dark palette, dark-mode Mermaid via Nextra's native rendering, working source-citation links, and the onboarding-first sidebar, without applying VitePress-specific Mermaid post-processing. (FR-008, FR-005)
3. **AC-003** — Where an adapter does not provide click-to-zoom or focus mode, the system shall complete the build successfully and declare those capabilities as absent. (FR-009)
4. **AC-004** — When no tool value is provided to `build` or `deploy`, the system shall select the VitePress adapter. (FR-010)
5. **AC-005** — When `deploy` runs for a chosen non-default tool, the system shall generate a workflow using that adapter's build command, output directory, and Node version from the manifest. (FR-011)
6. **AC-006** — The system shall not hardcode any tool's build command, output directory, or base-path logic in `build`, `deploy`, or CI. (FR-004)
7. **AC-007** — When a cross-cutting fix or palette change is applied to the core, the system shall apply it to every adapter without per-adapter edits. (FR-002)

## Success Criteria

- **SC-001**: Adding support for an additional documentation tool requires only a new adapter manifest (plus a theme delta), with no change to core packaging logic or to build/deploy control flow.
- **SC-002**: A before/after build of the same generated wiki with VitePress produces an equivalent site — same dark theme, Mermaid rendering, click-to-zoom, focus mode, and structure.
- **SC-003**: The same generated wiki builds successfully through both v1 adapters (VitePress and Nextra v4) at the declared parity bar.
- **SC-004**: No tool-specific build command, output directory, or base-path value appears anywhere outside an adapter manifest.

## Constraints

- VitePress behavior is unchanged — today's `/deep-wiki:build` output is the regression bar and the honesty check on the core's neutrality.
- No change to the wiki generation pipeline (catalogue → pages → onboarding); this track repackages existing generated Markdown only.
- Skills remain the portable source of truth; commands stay thin entrypoints.
- v1 ships exactly two adapters — VitePress (reference) and Nextra v4 (baseline parity).

## Out of Scope

- Fumadocs, Docusaurus, and Docus adapters — follow-up tracks against the proven contract.
- Starlight + `astro-mermaid` — the named first follow-up adapter (low-friction; client-side Mermaid, no headless browser), not built in v1.
- The generalized Mermaid-strategy slot (build-time pre-render beyond Nextra's native) and the multi-dialect Markdown normalizer.
- Auto-detect / content-aware tool recommendation — v1 uses explicit selection only.
- A golden-fixture parity CI harness and the no-build zero-dependency preview tier.

## Open Questions (deferred to planning)

- Exact manifest format and file layout (YAML vs skill frontmatter; where adapters live relative to `skills/wiki-vitepress/`).
- Core packaging shape: a new `wiki-site-core` skill versus a restructured `wiki-vitepress`, and how `build.md` / `deploy.md` reference it.
- How baseline parity is verified in v1 (manual check vs a minimal smoke build).
