---
name: deep-wiki-conventions
description: deep-wiki plugin documentation conventions — thin commands, skill delegation, reference file patterns, README accuracy
metadata:
  type: project
---

# deep-wiki Documentation Conventions

## Architecture (post-refactor on this branch)
- `commands/*.md` are thin entrypoints with `disable-model-invocation: true`
- Each thin command delegates to exactly one skill (the single source of truth)
- Skills live at `skills/<skill-name>/SKILL.md`
- `wiki-vitepress` skill additionally has `skills/wiki-vitepress/references/vitepress-build.md` as a sub-reference

## Delegation mapping (commands → skills)
| Command | Skill |
|---------|-------|
| ado.md | wiki-ado-convert |
| agents.md | wiki-agents-md |
| ask.md | wiki-qa |
| build.md | wiki-vitepress |
| catalogue.md | wiki-architect |
| changelog.md | wiki-changelog |
| llms.md | wiki-llms-txt |
| onboard.md | wiki-onboarding |
| page.md | wiki-page-writer |
| research.md | wiki-researcher |

## Orchestrator commands
- `commands/generate.md` and `commands/crisp.md` are NOT thin; they contain full procedure
- They cross-reference skills by prose: `the \`wiki-vitepress\` skill (\`references/vitepress-build.md\`)`
- `commands/build.md` uses a proper markdown link: `[references/vitepress-build.md](../skills/wiki-vitepress/references/vitepress-build.md)`

## Pre-existing README issue (not introduced by this diff)
- README line 46: Agents table lists `wiki-writer` — this is correct; refers to `agents/wiki-writer.md`
- README line 199-200: `wiki-vitepress/` tree shows only `SKILL.md`, omitting `references/vitepress-build.md` (added in this diff)
  — this is a gap but below the 80-confidence threshold since the file is discoverable via prose links

**Why:** Captured after reviewing the command-thinning refactor PR to preserve context for future reviews.
**How to apply:** When reviewing future PRs, check that commands delegate to the correct skill name and that orchestrator commands' cross-references resolve to existing skill paths.
