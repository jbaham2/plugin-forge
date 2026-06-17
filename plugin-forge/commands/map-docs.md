---
description: Stage 1 — interactively map a documentation site into a complete coverage roadmap.
argument-hint: <docs-url> [topic-slug]
allowed-tools: WebFetch, WebSearch, Read, Write, AskUserQuestion
---

Invoke the `map-docs` skill (plugin-forge Stage 1) to build an exhaustive documentation coverage map.

Arguments provided: `$ARGUMENTS`

- Treat the first argument as `<docs-url>` and the optional second as `[topic-slug]`.
- If no arguments were given, ask the user for the documentation URL before proceeding.
- Follow the `map-docs` skill exactly, including its interactive checkpoints, and write
  `<topic>-doc-roadmap.md`.
