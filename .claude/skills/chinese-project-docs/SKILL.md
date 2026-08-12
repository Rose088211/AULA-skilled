---
name: chinese-project-docs
description: Write and maintain accurate Chinese project documentation from repository evidence, including README.md, development guides, API notes, change summaries, and troubleshooting sections. Use when documenting a project for Chinese-speaking developers, translating technical project guidance, or updating docs after an implementation change.
---

# Chinese Project Docs

Keep documentation useful to the next developer: explain what the project does, how to run it, how to verify changes, and where important configuration lives. Treat code, manifests, scripts, and CI as authoritative over stale prose.

## Workflow

1. Inspect the repository and identify the audience, project status, supported environment, entry points, and confirmed commands. Do not document commands that were not verified or clearly mark them as examples.
2. Organize README content in this order when applicable: project purpose, current status, prerequisites, installation, local development, tests and checks, configuration, directory map, contribution workflow, and known limitations.
3. Use concise Chinese for explanations, but preserve code identifiers, filenames, commands, API paths, environment variable names, and error messages exactly. Use fenced code blocks for commands and examples.
4. Distinguish confirmed behavior, assumptions, and TODOs. Include expected output or success criteria for non-obvious commands. Never put real secrets, private URLs, or credentials in docs.
5. After editing, check Markdown structure, links, code fences, command accuracy, and `git diff --check`. Keep documentation changes scoped to the request.

## Style

Prefer concrete headings and short paragraphs. Explain decisions and constraints where they affect execution. Avoid marketing claims, filler sections, and translated jargon that makes commands harder to find.
