# Changelog

## [1.0.0] — 2026-05-20

### Added

- Initial release
- Fetch all PR review comments via GitHub REST and GraphQL APIs
- Three-way categorization: already-fixed / needs-fix / wontfix
- Automatic test runner detection (dotnet, npm, cargo, go, etc.)
- Auto-detect repo and PR number from git remote and current branch
- Reply to each review thread in Chinese via GraphQL `addPullRequestReviewThreadReply`
- Resolve threads after replying via GraphQL `resolveReviewThread`
- Commit messages with conventional commit format and Co-Authored-By trailer
- Project documentation update step (conditional, follows CLAUDE.md conventions)
- Bilingual README (English + Chinese)
- Reference docs for GitHub API endpoints and commit conventions
- Example fix patterns: fetch race condition, time range validation, thread-safe trimming, file writer cleanup
- GitHub issue templates (bug report, feature request) and PR template
