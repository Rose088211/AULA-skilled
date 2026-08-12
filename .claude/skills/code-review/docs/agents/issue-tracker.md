# Issue Tracker Reference

Use this template when a review references an issue, pull request, or external work item.

## Workflow

1. Extract issue identifiers from commit messages, branch names, or the review request.
2. Prefer the repository's configured issue-tracker CLI or API; do not invent issue details.
3. Record the source URL or identifier and quote the relevant requirement.
4. If no tracker access is configured, report the issue as unavailable and continue with the supplied specification.
5. Keep credentials in environment variables or the configured secret store; never write tokens into review artifacts.
