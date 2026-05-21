# Commit Message Conventions

## Format

```
<type>(<scope>): <short description in imperative mood>

Addresses review comments: #comment_id1, #comment_id2

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

## Review comment references

When a commit addresses specific review comments, include an "Addresses review
comments:" line after the subject line and before the optional body:

```
Addresses review comments: #12345, #12346
```

This creates a bidirectional trace: the commit message references the comment ID(s),
and the reply on each comment (posted by the `/pr-review-respond` skill) references
the commit SHA.

## Examples

```
fix(logger): prevent stale file writer accumulation

Addresses review comments: #12345

Clean up writers keyed by dates other than today to avoid
resource leaks in long-running processes.

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```
