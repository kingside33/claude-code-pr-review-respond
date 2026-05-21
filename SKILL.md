---
name: pr-review-respond
description: "Respond to all code review comments on a pull request, implement fixes, run tests, commit, reply on GitHub (in Chinese), and resolve conversations. Supports multi-round review loops with AI review bots. Use when the user says \"处理审查意见\", \"回复 review\", \"resolve PR comments\", \"修复审查意见\", \"处理PR评论\", \"review feedback\", \"address comments\", \"reply to review\", or similar phrases."
trigger: /pr-review-respond
version: 1.0.1
license: MIT
allowed-tools: Bash(gh *), Bash(git *), Read, Write, Edit, Grep, Glob
---

# /pr-review-respond

Handle all code review comments on a GitHub PR: categorize into fixed / needs-fix / wontfix, implement fixes, test, commit, push, reply in Chinese, and resolve all threads. Supports multi-round review loops to handle AI review bot feedback on pushed changes.

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

### Pre-Step — Initialize session state

At the start of every invocation, initialize:

- `round = 1`
- `max_rounds = 3` (configurable via environment variable `PR_REVIEW_MAX_ROUNDS`)
- `processed_thread_ids = []` — set of thread IDs already replied to and resolved in prior rounds
- `round_summary = []` — structured log of what each round accomplished

### Loop body: Steps 1–4

The four steps below form the body of the multi-round loop. After completing Step 4, a decision is made whether to continue to the next round.

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

If this is round 2 or later, filter the fetched threads:

1. Remove any thread whose `id` is already in `processed_thread_ids`
2. Remove any thread whose `isResolved` is true (resolved externally)
3. The remaining threads are "new unresolved threads" for this round

Map each remaining comment's `node_id` to its GitHub comment `id` (numeric) for cross-referencing in commit messages.

### Step 2 — Categorize each comment

For each top-level comment (not a reply):

| Category | Criteria | Round 1 Action | Round 2+ Action |
|----------|----------|----------------|-----------------|
| **Already fixed** | The fix exists in a prior commit (pre-existing or from a prior round) | Reply explaining which commit, then resolve | Reply with "已在第N轮修复：..." referencing the prior round's commit SHA |
| **Needs fix** | Valid issue, code change required | Fix → test → commit → push → reply → resolve | Same as round 1 |
| **Wontfix** | Invalid, out of scope, or design choice | Reply with rationale, then resolve | Same as round 1 |

For round 2+, when checking "already fixed": read the relevant file at the relevant lines. If the code already reflects the fix (i.e., the fix was applied in a prior round's commit), classify as "already fixed" and reference the prior round's commit SHA.

If after categorization there are zero "needs fix" and zero "already fixed" comments, exit the loop (nothing actionable this round).

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
4. Commit with a descriptive message following the format in "Commit message style".
   The "Addresses review comments:" line **must** include the comment ID(s) from each
   review comment addressed by this commit. Example:
   ```
   fix(logger): prevent stale file writer accumulation

   Addresses review comments: #12345, #12346

   Clean up writers keyed by dates other than today to avoid
   resource leaks in long-running processes.

   Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
   ```
   If one commit addresses multiple review comments, list all their IDs comma-separated.
5. Push

Group related fixes into sensible commits. Each commit should fix a cohesive set of issues and list all comment IDs it addresses.

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
- **Already fixed (first round)**: "已修复：<what was done>。Commit: <short SHA>（文件：<path>，对应评论 #id）"
- **Already fixed (subsequent rounds)**: "已在第N轮修复：<what was done>。Commit: <short SHA from prior round>（文件：<path>，对应评论 #id）"
- **Wontfix**: "不采纳：<reason>"
- Always mention the commit SHA (short form, 7 chars) if a fix was made
- Always append file path and comment ID reference: "（文件：<path>，对应评论 #id）"
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

### After processing this round's threads

1. For every thread replied to in this round, add its `thread.id` to `processed_thread_ids`
2. Record in `round_summary`:
   - Round number
   - All commit SHAs created in this round
   - Mapping of comment_id → commit_sha for each fix
   - List of wontfix comment IDs

### Loop exit / continue decision

After completing the loop body (Steps 1–4), decide whether to continue to the next round:

1. If `round >= max_rounds`: exit. Report "已达到最大轮次（{max_rounds}轮）。"
2. If zero commits were pushed this round (only wontfix replies): exit. No new code to review.
3. Otherwise, wait for AI review bot:
   a. Report: "等待AI审查机器人重新审查...（等待{interval}秒）"
   b. Sleep for `PR_REVIEW_POLL_INTERVAL` seconds (default 45, configurable via env var)
   c. Fetch fresh thread list via the same GraphQL query from Step 1
   d. Identify unresolved threads whose IDs are NOT in `processed_thread_ids`
   e. If none found: "未发现新的审查评论，退出循环。"
   f. If found: "发现 {N} 条新的审查评论，进入第 {round+1} 轮处理。" Increment `round` and go to Step 1.

### Post-Loop — Summary report

After the loop exits, compile and display a summary:

- **总完成轮次**: N
- **总提交数**: N
- **已修复的评论**: list of #comment_id → commit_sha mappings
- **不采纳的评论**: list of #comment_id
- **未处理的评论**: if any threads remain unresolved due to max_rounds, list them with a note suggesting the user re-run the skill or handle manually

### Step 6 — Update project documentation

If the project maintains a `CLAUDE.md` or similar convention file, follow its documentation update procedures. Common patterns include:

- `docs/change-log.md` — append fix summary to today's entry
- `docs/progress.md` — update recent progress and today's entry
- `CHANGELOG.md` — add entry under Unreleased

Read the relevant files first, then edit only the sections that need updating. If no documentation conventions exist, skip this step.

## Commit message style

Use this format:

```
<type>(<scope>): <short description in imperative mood>

Addresses review comments: #comment_id1, #comment_id2

<optional body explaining WHY>

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

The "Addresses review comments:" line is **required** for any commit that fixes review feedback. It lists the GitHub comment IDs addressed by this commit, separated by commas.

Types: `fix` (default), `test`, `refactor`, `chore`

## Key rules

1. **Minimal changes** — fix only what the review comment asks for, no extra refactoring
2. **Test before commit** — always run relevant tests, confirm pass
3. **Chinese replies** — all GitHub comment replies must be in Chinese
4. **Group related fixes, annotate comment IDs** — combine related fixes into sensible commits (don't create a commit per comment). Each commit's message MUST include an "Addresses review comments:" line listing the comment ID(s) it addresses.
5. **Reply under each specific comment** — DO NOT add a general summary comment at the PR level. Reply individually under each review thread using `addPullRequestReviewThreadReply`.
6. **Reply then resolve immediately** — process one thread at a time: post reply, then immediately resolve that thread. Do NOT batch resolve all threads at the end.
7. **Multi-round loop** — after resolving all threads in a round, wait for the AI review bot to post new comments (45s default, configurable via `PR_REVIEW_POLL_INTERVAL`). Process up to `max_rounds` rounds (default 3). If no new unresolved threads appear, exit early. Log a summary of all rounds at the end.
8. **Bidirectional referencing** — every fix commit message must reference the comment IDs it addresses (via "Addresses review comments:" line). Every reply about a fix must reference the commit SHA, file path, and comment ID. This creates a complete traceability chain: comment ↔ commit ↔ reply.
9. **Don't push without testing** — if tests fail, fix the issue and re-test
10. **Resolve ALL threads** — even wontfix threads must be resolved after replying

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PR_REVIEW_POLL_INTERVAL` | `45` | Seconds to wait for AI review bot to post new comments between rounds |
| `PR_REVIEW_MAX_ROUNDS` | `3` | Maximum number of review rounds before stopping |

## References

- GitHub API endpoints: see [`references/github-api.md`](references/github-api.md)
- Commit conventions: see [`references/commit-conventions.md`](references/commit-conventions.md)
- Project-specific fix patterns: see [`examples/`](examples/)