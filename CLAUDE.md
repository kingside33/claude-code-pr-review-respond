# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## What this is

A single-file Claude Code skill (`SKILL.md`) that automates responding to
code review comments on GitHub PRs. No build system, no dependencies, no
tests — the skill is a markdown file consumed by Claude Code.

## Repository structure

- `SKILL.md` — the skill itself. This is the product.
- `README.md` / `README.zh-CN.md` — public-facing bilingual documentation
- `CHANGELOG.md` — version history
- `examples/` — project-specific fix patterns extracted from SKILL.md
- `references/` — supplementary API and convention references

## How to make changes

Edit `SKILL.md` directly. There's nothing to build or test. When making changes:

- Bump the version in SKILL.md frontmatter (`version: X.Y.Z`)
- Add a dated entry to CHANGELOG.md
- Update README.md / README.zh-CN.md if features, usage, or requirements change
- Keep README feature list in sync with actual SKILL.md content

## Key constraints

- The skill must remain generic — no project-specific test commands or patterns
- GitHub API patterns (GraphQL + REST) must work with `gh` CLI
- All replies must be in Chinese
- Commit messages must follow conventional commit format with Co-Authored-By
