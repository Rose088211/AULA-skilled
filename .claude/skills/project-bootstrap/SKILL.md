---
name: project-bootstrap
description: Inspect an unfamiliar repository, identify its stack, package manager, entry points, development commands, and current risks, then establish a concise evidence-based workflow. Use when starting work in a repository, onboarding an agent, the project structure is unclear, or another agent is already modifying the workspace.
---

# Project Bootstrap

Build a reliable working model of the repository before changing code. Treat files and command output as the source of truth; do not infer a framework or invent commands from directory names alone.

## Workflow

1. Inspect the workspace:
   - Run `git status --short --branch` and note pre-existing changes.
   - List top-level files and inspect the README, manifests, lockfiles, configuration, and CI files.
   - Exclude generated and dependency directories from broad searches.
2. Identify the project contract:
   - Detect the language and framework from manifests and source imports.
   - Identify the package manager from the lockfile and prefer its declared scripts.
   - Find the application entry point, test roots, build output, environment templates, and deployment configuration.
   - Check `AGENTS.md`, `CLAUDE.md`, `.claude/`, `.agents/`, and contribution instructions.
3. Establish safe next steps:
   - Report confirmed commands for install, lint, typecheck, test, build, and local run.
   - Separate confirmed facts from assumptions and missing prerequisites.
   - Identify untracked files, secrets risk, large generated artifacts, and concurrent-agent changes.
   - Before editing, state the smallest useful next action and its expected evidence.

## Concurrent Work

When another agent is active, do not reset, clean, checkout, or overwrite its changes. Re-read `git status` immediately before edits, keep ownership boundaries explicit, and report conflicts instead of silently resolving unrelated modifications. Prefer additive, scoped files until the project structure is understood.

## Output

Return a short inventory containing:

- Stack and package manager
- Important paths
- Confirmed commands
- Current workspace state
- Risks, assumptions, and the next action
