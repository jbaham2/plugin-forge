---
description: Stage 3 — execute a filled <topic>-plugin-prompt.md to build the actual Claude Code plugin.
argument-hint: [path-to-plugin-prompt.md]
allowed-tools: Read, Write, Edit, Bash, WebFetch, WebSearch, Skill, Glob, Grep, Agent
---

Invoke the `build-plugin` skill (plugin-forge Stage 3) to build the target plugin from a filled
builder prompt.

Arguments provided: `$ARGUMENTS`

- Treat the argument as the path to a filled `<topic>-plugin-prompt.md`. If none was given, look for
  a `*-plugin-prompt.md` in the working directory; if several exist, ask which to use.
- Precondition: a filled prompt must exist. If the user only has docs and no filled prompt, run
  `/plugin-forge:forge` or `/plugin-forge:build-prompt` first.
- Follow the `build-plugin` skill exactly: confirm the target build directory, then run the embedded
  prompt's start protocol and wait for approval before scaffolding.
