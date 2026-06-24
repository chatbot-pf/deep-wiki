# Workflow

Task lifecycle, quality gates, and conventions for working in this repo.

## Standard task lifecycle

1. Pick the next task from the track's `plan.md`.
2. Make the change (edit skills/commands/agents Markdown, or scaffold/adjust generated-site config).
3. Validate (see Quality Gates).
4. Commit per task using Conventional Commits.
5. Record progress in the track's living plan sections.

## Quality gates

This is a prompt/Markdown plugin, so "tests" are validation passes rather than a unit-test suite:

- **Skill/command coherence.** A command's behavior lives in its skill/reference; entrypoints stay thin. Cross-runtime behavior stays in the skill, not the command.
- **Generated-site smoke build.** For packaging changes, the acceptance bar is that a representative wiki builds into a valid static site with Mermaid rendering in dark mode and intact source citations (`cd wiki && npm install && npm run build`). For multi-tool work, every supported adapter must pass this on the same fixture.
- **No regressions to existing output.** VitePress `/deep-wiki:build` output stays behavior-identical unless a change explicitly targets it.
- **Citations preserved.** Generated content keeps `file:line` source links.

## Commit convention

Conventional Commits, imperative mood, ≤100-char header. Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `build`, `ci`. One logical change per commit; commit per task.

## Development commands

- Install generated-site deps: `cd wiki && npm install`
- Build generated site: `cd wiki && npm run build`
- Preview generated site: `cd wiki && npm run preview`
- Run please plugin scripts: `bun run <script>`

## Branch & PR

- Single PR per track (stacked-PR workflow disabled). `/please:implement` creates the feature branch; `/please:finalize` opens the PR.
