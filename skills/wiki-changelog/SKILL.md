---
name: wiki-changelog
description: Analyzes git commit history and generates structured changelogs categorized by change type. Use when the user asks about recent changes, wants a changelog, or needs to understand what changed in the repository.
license: MIT
user-invocable: false
metadata:
  author: Microsoft
  version: "1.0.0"
---

# Wiki Changelog

Generate structured changelogs from git history.

## Source Repository Resolution (MUST DO FIRST)

Before generating any changelog, you MUST determine the source repository context:

1. **Check for git remote**: Run `git remote get-url origin` to detect if a remote exists
2. **Ask the user**: _"Is this a local-only repository, or do you have a source repository URL (e.g., GitHub, Azure DevOps)?"_
   - Remote URL provided → store as `REPO_URL`, use **linked citations** for commit hashes and file references
   - Local-only → use plain commit hashes and file references
3. **Do NOT proceed** until source repo context is resolved

## When to Activate

- User asks "what changed recently", "generate a changelog", "summarize commits"
- User wants to understand recent development activity

## Procedure

1. Examine git log (commits, dates, authors, messages)
2. Group by time period: daily (last 7 days), weekly (older)
3. Classify each commit: Features (🆕), Fixes (🐛), Refactoring (🔄), Docs (📝), Config (🔧), Dependencies (📦), Breaking (⚠️)
4. Generate concise user-facing descriptions using project terminology

## Categories

| Emoji | Category | Signal Keywords |
|-------|----------|----------------|
| 🆕 | New Features | `feat`, `add`, `new`, `implement`, `introduce` |
| 🐛 | Bug Fixes | `fix`, `bug`, `patch`, `resolve`, `hotfix` |
| 🔄 | Refactoring | `refactor`, `restructure`, `reorganize`, `clean` |
| 📝 | Documentation | `docs`, `readme`, `comment`, `jsdoc`, `docstring` |
| 🔧 | Configuration | `config`, `env`, `setting`, `ci`, `build` |
| 📦 | Dependencies | `deps`, `upgrade`, `bump`, `package`, `lock` |
| ⚠️ | Breaking Changes | `breaking`, `BREAKING`, `migrate`, `deprecate` |

## Output

For each time period, output:

```markdown
## [Date or Date Range]

**[Summary Title]**

[1-2 sentence overview]

### 🆕 New Features
- [Change description]

### 🐛 Bug Fixes
- [Change description]

### ⚠️ Breaking Changes
- [Change description with migration notes]
```

## Constraints

- Focus on user-facing changes
- Merge related commits into coherent descriptions
- Use project terminology from README
- Highlight breaking changes prominently with migration notes
- When `REPO_URL` is available, link commit hashes: `[abc1234](REPO_URL/commit/abc1234)` and changed files: `[file_path](REPO_URL/blob/BRANCH/file_path)`
