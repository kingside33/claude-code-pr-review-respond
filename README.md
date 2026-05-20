# claude-code-pr-review-respond

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)](SKILL.md)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-orange.svg)](https://claude.ai/code)

> [中文版本](README.zh-CN.md)

A [Claude Code](https://claude.ai/code) skill that automates responding to code review comments on GitHub pull requests. It fetches review comments, categorizes them, implements fixes, runs tests, commits, pushes, replies in Chinese, and resolves all conversation threads.

## What it does

1. **Fetches** all PR review comments via GitHub API (GraphQL + REST)
2. **Categorizes** each comment into: _already fixed_ / _needs fix_ / _wontfix_
3. **Implements** code fixes for "needs fix" items, runs tests, commits, pushes
4. **Replies** to each review thread in Chinese (中文) on GitHub
5. **Resolves** each conversation thread after replying

## Why a skill, not just a prompt

- Systematic categorization of review comments rather than ad-hoc handling
- Proper GitHub GraphQL API usage for thread-level replies and resolution
- Consistent Chinese-language reply format
- Commit message conventions with Co-Authored-By attribution
- Test-before-commit enforcement
- Auto-detection of test runners across tech stacks

## Installation

### Option 1: Git Clone (Recommended)

```bash
git clone https://github.com/kingside33/claude-code-pr-review-respond \
  ~/.claude/skills/pr-review-respond
```

Then reference it in `~/.claude/CLAUDE.md`:

```markdown
- **pr-review-respond** (`~/.claude/skills/pr-review-respond/SKILL.md`) — PR review response automation. Trigger: `/pr-review-respond`
```

### Option 2: npx skills add

```bash
npx skills add https://github.com/kingside33/claude-code-pr-review-respond
```

### Option 3: Manual Download

Download `SKILL.md` to any location and add to `CLAUDE.md`:

```markdown
- PR review response → read `~/skills/pr-review-respond/SKILL.md`
```

## Usage

```
/pr-review-respond <owner/repo> <PR number>
```

If no arguments are given, the skill auto-detects from `git remote get-url origin` and the current branch's associated PR.

## Requirements

- [GitHub CLI](https://cli.github.com/) (`gh`) installed and authenticated (`gh auth login`)
- Git repository with a GitHub remote
- Write access to the repository
- Project must have a test suite

## How it works

### Step 1 — Fetch review comments

Uses `gh api` to retrieve all PR review comments (REST) and thread resolution status (GraphQL).

### Step 2 — Categorize

| Category | Criteria | Action |
|----------|----------|--------|
| Already fixed | Fix exists in a prior commit | Reply with commit ref, resolve |
| Needs fix | Valid issue requiring code change | Fix → test → commit → push → reply → resolve |
| Wontfix | Invalid, out of scope, or design choice | Reply with rationale, resolve |

### Step 3 — Implement fixes

Makes minimal code changes, auto-detects and runs the test suite, commits with conventional commit format including `Co-Authored-By` trailer, then pushes.

### Step 4 — Reply and resolve

Posts a reply in Chinese under each review thread using GraphQL, then immediately resolves that thread. Processes one thread at a time.

### Step 5 — Update docs

Optionally updates project documentation (`docs/change-log.md`, `docs/progress.md`, etc.) if the project maintains them according to CLAUDE.md conventions.

## Reply format

All replies are in Chinese:

- **Already fixed**: "已修复：\<description\>。Commit: \<short SHA\>"
- **Wontfix**: "不采纳：\<reason\>"
- **Needs fix (after fixing)**: "已修复：\<description\>。Commit: \<short SHA\>"

## Repository structure

```
.
├── SKILL.md                # Core skill definition
├── README.md               # English documentation
├── README.zh-CN.md         # Chinese documentation
├── LICENSE                 # MIT License
├── CHANGELOG.md            # Version history
├── CONTRIBUTING.md         # Contribution guide
├── examples/               # Project-specific fix patterns
│   ├── fetch-race-condition.md
│   ├── time-range-validation.md
│   ├── thread-safe-trimming.md
│   └── file-writer-cleanup.md
├── references/             # Supplementary references
│   ├── github-api.md
│   └── commit-conventions.md
└── .github/                # Issue and PR templates
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE) for details.
