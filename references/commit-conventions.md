# Commit Message Conventions

## Format

```
<type>(<scope>): <short description in imperative mood>

<optional body explaining WHY>

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

## Types

| Type | Usage |
|------|-------|
| `fix` | Bug fix (default for review response commits) |
| `test` | Adding or updating tests |
| `refactor` | Code restructuring without behavior change |
| `chore` | Maintenance tasks, dependency updates |

## Scope

Optional context in parentheses, e.g.:

- `fix(auth): handle expired token gracefully`
- `test(api): add time range validation`

## Co-author trailer

Every commit must include:

```
Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

This attributes AI-assisted changes per Anthropic's guidelines.

## Examples

```
fix(logger): prevent stale file writer accumulation

Clean up writers keyed by dates other than today to avoid
resource leaks in long-running processes.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```
