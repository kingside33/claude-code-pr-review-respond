---
name: pr-review-respond
description: "Respond to all code review comments on a pull request, implement fixes, run tests, commit, reply on GitHub (in Chinese), and resolve conversations. Use when the user says \"处理审查意见\", \"回复 review\", \"resolve PR comments\", \"修复审查意见\", \"处理PR评论\", \"review feedback\", \"address comments\", \"reply to review\", or similar phrases."
trigger: /pr-review-respond
version: 1.0.0
license: MIT
allowed-tools: Bash(gh *), Bash(git *), Read, Write, Edit, Grep, Glob
---

# /pr-review-respond

Handle all code review comments on a GitHub PR: categorize into fixed / needs-fix / wontfix, implement fixes, test, commit, push, reply in Chinese, and resolve all threads.

## When to use

- A PR has pending review comments that need to be addressed
- The user says "处理审查意见", "回复 review", "resolve PR comments", etc.
- Review feedback from any review tool (codereviewbot, human reviewers, etc.)

## When NOT to use

- The PR has no review comments yet — nothing to respond to
- You don't have write access to the repository
- The review comments require discussion with the original author before acting
- The skill is being invoked on a fork without upstream write permissions

## Usage

```
/pr-review-respond <owner/repo> <PR number>
```

If no arguments given, detect from `git remote get-url origin` and the current branch's associated PR:

```bash
# Auto-detect PR number from current branch
gh pr view --json number --jq '.number'
```

## Workflow

### Step 1 — Fetch all review comments

Use `gh api` to get all pull request review comments:

```bash
gh api repos/<owner>/<repo>/pulls/<PR>/comments --paginate \
  --jq '.[] | {id, body: .body[0:100], path, commit_id, user: .user.login, created_at, in_reply_to_id, node_id}'
```

Also fetch review threads to check what's already resolved:

```bash
gh api graphql -f query='
query($owner:String!,$repo:String!,$pr:Int!) {
  repository(owner:$owner,name:$repo) {
    pullRequest(number:$pr) {
      reviewThreads(first:100) {
        nodes { id isResolved comments(first:1) { nodes { id } } }
      }
    }
  }
}' -F owner=<owner> -F repo=<repo> -F pr=<PR>
```

### Step 2 — Categorize each comment

For each top-level comment (not a reply):

| Category | Criteria | Action |
|----------|----------|--------|
| **Already fixed** | The fix exists in a prior commit | Reply explaining which commit, then resolve |
| **Needs fix** | Valid issue, code change required | Fix → test → commit → push → reply → resolve |
| **Wontfix** | Invalid, out of scope, or design choice | Reply with rationale, then resolve |

### Step 3 — Implement fixes

For each "needs fix" comment:

1. Read the relevant file(s)
2. Make the minimal code change
3. Run the project's test suite. Auto-detect the test runner from:
   - `package.json` scripts (e.g. `npm test`, `npx vitest`, `npx jest`)
   - `*.csproj` / `*.sln` (use `dotnet test`)
   - `Makefile` test targets
   - `Cargo.toml` (use `cargo test`)
   - `go.mod` (use `go test ./...`)
   - If the test command is unclear, ask the user which command to use.
4. Commit with a descriptive message, ending with:
   ```
   Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
   ```
5. Push

Group related fixes into sensible commits. Each commit should fix a cohesive set of issues.

### Step 4 — Reply to each comment on GitHub and resolve

For each review thread, reply individually using GraphQL:

```bash
gh api graphql -f query='
mutation {
  addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "<thread_id>",
    body: "<reply text in Chinese>"
  }) {
    comment { id body }
  }
}'
```

**Reply format rules:**
- **Already fixed**: "已修复：<what was done>。Commit: <short SHA>"
- **Wontfix**: "不采纳：<reason in Chinese>"
- Always mention the commit SHA (short form) if a fix was made
- Always reply in Chinese (中文)
- Keep replies concise — one to two sentences
- **DO NOT add a general summary comment at the PR level** — reply only under each specific comment thread

After replying to a thread, resolve it immediately via GraphQL:

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: { threadId: "<thread_id>" }) {
    thread { id }
  }
}'
```

Process one thread at a time: reply → resolve → next thread. Do NOT batch resolve all threads at the end.

### Step 5 — Update project documentation

If the project maintains a `CLAUDE.md` or similar convention file, follow its documentation update procedures. Common patterns include:

- `docs/change-log.md` — append fix summary to today's entry
- `docs/progress.md` — update recent progress and today's entry
- `CHANGELOG.md` — add entry under Unreleased

Read the relevant files first, then edit only the sections that need updating. If no documentation conventions exist, skip this step.

## Commit message style

Use this format:
```
<type>(<scope>): <short description in imperative mood>

<optional body explaining WHY>

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

Types: `fix` (default), `test`, `refactor`, `chore`

## Key rules

1. **Minimal changes** — fix only what the review comment asks for, no extra refactoring
2. **Test before commit** — always run relevant tests, confirm pass
3. **Chinese replies** — all GitHub comment replies must be in Chinese
4. **One commit per logical group** — don't create a commit per comment, group related fixes
5. **Reply under each specific comment** — DO NOT add a general summary comment at the PR level. Reply individually under each review thread using `addPullRequestReviewThreadReply`.
6. **Reply then resolve immediately** — process one thread at a time: post reply, then immediately resolve that thread. Do NOT batch resolve all threads at the end.
7. **Don't push without testing** — if tests fail, fix the issue and re-test
8. **Resolve ALL threads** — even wontfix threads must be resolved after replying

## References

- GitHub API endpoints: see [`references/github-api.md`](references/github-api.md)
- Commit conventions: see [`references/commit-conventions.md`](references/commit-conventions.md)
- Project-specific fix patterns: see [`examples/`](examples/)
