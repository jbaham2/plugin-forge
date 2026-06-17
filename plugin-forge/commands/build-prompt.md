---
description: Stage 2 — research a service and fill the plugin-builder template into a runnable prompt.
argument-hint: <docs-url> [topic-slug]
allowed-tools: WebFetch, WebSearch, Read, Write
---

Invoke the `build-prompt` skill (plugin-forge Stage 2) to research the service and produce a filled,
runnable plugin-builder prompt.

Arguments provided: `$ARGUMENTS`

- Treat the first argument as `<docs-url>` and the optional second as `[topic-slug]`.
- If no arguments were given, ask the user for the documentation URL before proceeding.
- Follow the `build-prompt` skill exactly, including Step 0 (reuse an existing `<topic>-doc-roadmap.md`
  if present instead of re-crawling), and write `<topic>-plugin-prompt.md`.
