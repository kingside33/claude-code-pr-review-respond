# Changelog

## [1.0.2] — 2026-05-21

### Fixed

- Step 2 premature exit: wontfix comments are now replied to and resolved before exiting the loop, instead of being silently skipped
- Step numbering gap: "Post-Loop — Summary report" renumbered as "Step 5 — Summary report" for consistency

### Added

- GraphQL `reviewThreads(first:100)` pagination note documenting the limit and cursor-based pagination for PRs with >100 threads
- Error handling guidance: if `resolveReviewThread` fails (thread already resolved externally), continue to next thread gracefully

## [1.0.1] — 2026-05-21

### Added

- Multi-round review loop: after pushing fixes, automatically wait for AI review bot feedback (default 45s, configurable via `PR_REVIEW_POLL_INTERVAL`) and process new comments, up to 3 rounds (configurable via `PR_REVIEW_MAX_ROUNDS`)
- Bidirectional traceability between commits and review comments: commit messages now include `Addresses review comments: #id` line
- Reply format enhanced to include file path and comment ID reference
- Post-loop summary report showing all rounds' commit-to-comment mappings
- Environment variables section documenting `PR_REVIEW_POLL_INTERVAL` and `PR_REVIEW_MAX_ROUNDS`

### Changed

- Restructured workflow from linear single-pass to multi-round loop with exit/continue logic
- Round 2+ categorization includes round-number-aware "already fixed" handling
- Key rules updated: rule 4 now requires comment ID annotation, added rules 7-8 for multi-round loop and bidirectional referencing
- Updated commit-conventions reference docs with `Addresses review comments:` format

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
