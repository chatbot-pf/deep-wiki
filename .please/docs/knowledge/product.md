# Product Guide

## What this is

**deep-wiki** is an AI-powered wiki generator for code repositories, packaged as an Agent Skills plugin (GitHub Copilot CLI / Claude Code, and other Agent Skills runtimes such as Codex and Antigravity). It turns any codebase into comprehensive, structured, Mermaid-rich documentation — with dark-mode static sites, audience-tailored onboarding guides, changelogs, and deep research — all grounded in source-linked citations.

It is a fork of the `deep-wiki` plugin from [microsoft/skills](https://github.com/microsoft/skills), distilled from the prompt architectures of OpenDeepWiki and deepwiki-open. Licensed MIT.

## Target users

- **Developers and maintainers** who need accurate, navigable documentation of a codebase without writing it by hand.
- **New contributors, staff engineers, executives, and PMs** consuming audience-tailored onboarding guides.
- **Coding agents and MCP tools** that discover and consume the generated `llms.txt`, `AGENTS.md`, and wiki pages.

## Core outcome

A reader (human or agent) can understand an unfamiliar codebase quickly: big-picture architecture first, then drill-down detail, every claim traceable to `file:line`, with diagrams that make structure legible.

## Capabilities (commands)

Wiki generation (`generate`, `crisp`, `catalogue`, `page`), knowledge artifacts (`onboard`, `changelog`, `research`, `ask`, `agents`, `llms`), and packaging/distribution (`build` → static site, `deploy` → GitHub Pages, `ado` → Azure DevOps Wiki). Auto-invoked skills (`wiki-architect`, `wiki-page-writer`, `wiki-researcher`, `wiki-vitepress`, etc.) back these commands.

## Direction

The packaging layer is currently VitePress-only and deeply coupled to it. An active direction is making site packaging **multi-tool** behind a generator-neutral core + adapter contract (see `docs/brainstorms/` and `docs/ideation/`), so the wiki can target VitePress, Nextra, Starlight, and others without forking the build per tool.
