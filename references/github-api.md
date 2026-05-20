# GitHub API Reference

Endpoints and GraphQL mutations used by the `pr-review-respond` skill.

## REST API

### List review comments on a pull request

```bash
gh api repos/<owner>/<repo>/pulls/<PR>/comments --paginate \
  --jq '.[] | {id, body: .body[0:100], path, commit_id, user: .user.login, created_at, in_reply_to_id, node_id}'
```

Returns all review comments (both inline and general) on the PR.
Use `--paginate` to fetch all pages automatically.

## GraphQL API

### Fetch review threads and their resolution status

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

Returns thread IDs and whether each is already resolved.

### Reply to a review thread

```bash
gh api graphql -f query='
mutation {
  addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "<thread_id>",
    body: "<reply text>"
  }) {
    comment { id body }
  }
}'
```

Posts a reply under the specific review comment thread (not a general PR comment).

### Resolve a review thread

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: { threadId: "<thread_id>" }) {
    thread { id }
  }
}'
```

Marks a review thread as resolved.

## Authentication

All commands require `gh` CLI to be installed and authenticated:

```bash
gh auth login
```

The authenticated user must have write access to the repository.
