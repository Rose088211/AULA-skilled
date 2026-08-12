---
name: release-readiness
description: Validate a change before commit, code review, merge, deployment, or release by inspecting the diff, repository instructions, tests, build evidence, secrets risk, and generated artifacts. Use when someone asks whether work is ready, requests a final check, or another agent has completed a change.
---

# Release Readiness

Produce evidence for readiness; do not declare success from a clean-looking diff alone. Match validation depth to the changed surface and preserve unrelated work already in the repository.

## Checklist

1. Establish scope with `git status --short --branch`, `git diff --stat`, `git diff`, and a review of untracked files. Read relevant `CLAUDE.md`, `AGENTS.md`, and project instructions first.
2. Map each changed area to validation: unit tests for logic, integration tests for boundaries, typecheck/lint for source changes, build for packaging, and browser tests or screenshots for UI changes.
3. Run only commands confirmed by the repository. If a check cannot run, record the exact prerequisite or failure instead of substituting an unrelated command.
4. Scan changed text files for accidental secrets, private endpoints, local paths, debug output, and generated dependency churn. Do not print secret values.
5. Re-check status after validation. Never reset or clean the workspace. Do not commit, push, deploy, or publish unless the user explicitly requests that separate action.

## Verdict

Report:

- `Ready`, `Ready with caveats`, or `Blocked`.
- Checks run, with commands and pass/fail results.
- Findings ordered by severity, with file references.
- Tests or checks not run and why.
- Exact remaining action for the user or owning agent.
