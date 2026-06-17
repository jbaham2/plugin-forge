---
description: Run the full docs→plugin pipeline (map-docs → build-prompt → build-plugin) from a documentation URL.
argument-hint: <docs-url> [topic-slug]
allowed-tools: Read, Write, WebFetch, WebSearch, Skill, Bash, Glob, Grep, Agent, AskUserQuestion
---

Invoke the `forge` skill (plugin-forge Stage orchestrator) to run the complete documentation→plugin
pipeline end-to-end.

Arguments provided: `$ARGUMENTS`

- Treat the first argument as `<docs-url>` and the optional second as `[topic-slug]`.
- If no arguments were given, ask the user for the documentation URL before proceeding.
- Follow the `forge` skill's orchestration exactly: confirm scope, then run Stage 1 (`map-docs`),
  Stage 2 (`build-prompt`), and Stage 3 (`build-plugin`), pausing for confirmation between stages.
