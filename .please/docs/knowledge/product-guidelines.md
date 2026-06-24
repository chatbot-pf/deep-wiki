# Product Guidelines

These are the design principles the generated output must honor. They come from the project README's "Design Principles" and govern both the prompts/skills and any new packaging work.

## Content principles

1. **Source-linked citations.** Every claim cites `file_path:line_number`. Remote repos use `[file:line](REPO_URL/blob/BRANCH/file#Lline)`; local repos use `(file:line)`. Never hand-wave.
2. **Structure-first.** Always generate a TOC/catalogue before page content.
3. **Evidence-based / never invent.** All content is derived from actual code — trace real implementations; never guess from file names.
4. **Diagram-rich.** Minimum 3–5 dark-mode Mermaid diagrams per page, multiple diagram types, with `<!-- Sources: ... -->` blocks. More diagrams is better.
5. **Table-driven.** Prefer tables over prose for structured information; always include a "Source" column with citations.
6. **Progressive disclosure.** Big picture first, then detail; every section starts with a TL;DR.
7. **Hierarchical depth.** Max 4 levels (Architecture → Subsystems → Components → Methods).
8. **Depth before breadth.** Trace actual code paths.

## Output and rendering

9. **Dark-mode native.** All output is designed for dark-theme rendering. The canonical palette (`#0d1117` background, `#6d5dfc` accent, `#e6edf3` text, `#161b22`/`#2d333b`/`#1c2333` surfaces) is used everywhere and should be treated as design tokens, not duplicated literals.
10. **Agent-discoverable.** Place output at standard paths (`llms.txt` at repo root, `AGENTS.md` in key folders) so coding agents and MCP tools find it automatically.

## Engineering guidelines for this repo

- The "code" of this plugin is Markdown: `skills/*/SKILL.md` (+ `references/`), `commands/*.md`, `agents/*.md`. Keep skills as the portable source of truth; commands are thin entrypoints that delegate to skills.
- When a behavior must work across runtimes (Copilot CLI, Codex, Antigravity), keep it in the skill/reference, not the command.
- Prefer extracting tool-independent logic into shared references over duplicating it per tool.
