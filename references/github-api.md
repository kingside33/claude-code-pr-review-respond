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

### Fetch review threads and their resolution status (with pagination)

```bash
# Paginates through all review threads via cursor-based pagination
all_threads="[]"
cursor="null"
has_next_page="true"

while [ "$has_next_page" = "true" ]; do
  page=$(gh api graphql -f query='
  query($owner:String!,$repo:String!,$pr:Int!,$after:String) {
    repository(owner:$owner,name:$repo) {
      pullRequest(number:$pr) {
        reviewThreads(first:100, after:$after) {
          pageInfo { hasNextPage endCursor }
          nodes { id isResolved comments(first:1) { nodes { id } } }
        }
      }
    }
  }' -F owner=<owner> -F repo=<repo> -F pr=<PR> ${cursor:+-F after="$cursor"})

  page_nodes=$(echo "$page" | jq '.data.repository.pullRequest.reviewThreads.nodes')
  all_threads=$(echo "$all_threads" | jq --argjson nodes "$page_nodes" '. + $nodes')
  has_next_page=$(echo "$page" | jq -r '.data.repository.pullRequest.reviewThreads.pageInfo.hasNextPage')
  cursor=$(echo "$page" | jq -r '.data.repository.pullRequest.reviewThreads.pageInfo.endCursor')
done
```

Returns all thread IDs and whether each is already resolved, with no limit on thread count.

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

## Retry and error handling

All `gh api` calls should implement this retry pattern:

1. Execute the command, capturing stdout and stderr
2. If the command exits with non-zero or returns invalid JSON:
   - If the error contains `rate limit`, `403`, or `429`: wait 60 seconds and retry
   - Otherwise: retry up to N times with 1-second exponential backoff
3. If all retries fail: report the error and abort

After each successful call, check remaining rate limit:
```bash
remaining=$(gh api rate_limit --jq '.rate.remaining' 2>/dev/null)
```
If `remaining < 100`, log a warning. If `remaining < 10`, pause before the next API call.

## Authentication

All commands require `gh` CLI to be installed and authenticated:

```bash
gh auth login
```

The authenticated user must have write access to the repository.
