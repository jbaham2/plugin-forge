---
name: forge
description: This skill should be used when the user asks to "forge a plugin from these docs", "turn this documentation into a Claude Code plugin", "run the full plugin pipeline", "go from docs URL to plugin", or gives a documentation link and wants the entire docs→plugin workflow run end-to-end. Orchestrates the three plugin-forge stages in sequence.
argument-hint: <docs-url> [topic-slug]
allowed-tools: Read, Write, WebFetch, WebSearch, Skill, Bash, Glob, Grep, Agent, AskUserQuestion
version: 0.1.0
---

# Forge — docs → expert plugin, end to end

Orchestrate the full plugin-forge pipeline: take a documentation URL and walk through all three
stages, handing artifacts between them. Use the individual stage skills (`map-docs`, `build-prompt`,
`build-plugin`) for à-la-carte runs; use `forge` to run the whole thing.

## Inputs

- `<docs-url>` (required) — the documentation root or website.
- `[topic-slug]` (optional) — kebab-case slug; derived from the product name if omitted.

## The pipeline

```
Stage 1: map-docs      <docs-url>            →  <topic>-doc-roadmap.md
Stage 2: build-prompt  <docs-url> (+roadmap) →  <topic>-plugin-prompt.md
Stage 3: build-plugin  <topic>-plugin-prompt.md  →  ./<topic>-plugin/  (the built plugin)
```

Each stage's output is the next stage's input. Confirm with the user between stages — each stage
is substantial and the user may want to inspect or edit an artifact before proceeding.

## How to orchestrate

1. **Stage 1 — Map the docs.** Invoke the `map-docs` skill with `<docs-url>` (and `[topic-slug]`).
   Run its interactive checkpoints. Output: `<topic>-doc-roadmap.md`. Then pause and ask the user
   to confirm the roadmap looks complete before continuing.
2. **Stage 2 — Build the prompt.** Invoke the `build-prompt` skill with the same `<docs-url>`. It
   detects the roadmap from Stage 1 and uses it instead of re-crawling. Output:
   `<topic>-plugin-prompt.md`. Pause and let the user review/adjust the derived placeholders and
   source tracker (especially any `⚠ inferred — confirm` items).
3. **Stage 3 — Build the plugin.** Invoke the `build-plugin` skill with `<topic>-plugin-prompt.md`.
   Confirm the target build directory (default `./<topic>-plugin`), then run the embedded build
   protocol's start protocol and proceed on approval.

## Orchestration principles

- **Stage 1 is recommended but skippable.** If the user wants to skip mapping, go straight to
  Stage 2 — `build-prompt` will self-crawl the docs (less exhaustive). Offer this choice up front.
- **Carry the `<topic>` slug across stages** so filenames line up
  (`<topic>-doc-roadmap.md` → `<topic>-plugin-prompt.md` → `<topic>-plugin/`).
- **Stop on failure.** If a stage produces a weak or incomplete artifact (e.g. many `[UNVERIFIED]`
  claims or sparse coverage), surface it and let the user fix or rerun that stage before advancing.
- **Don't duplicate stage logic.** Delegate to each stage skill rather than re-implementing it here;
  this skill only sequences and hands off.

## Definition of done

- All requested stages ran in order, each consuming the prior stage's artifact.
- The user confirmed (or explicitly skipped) the checkpoint between each stage.
- The final artifact for the requested scope exists: a roadmap, a filled prompt, and/or a built
  plugin, depending on how far the user chose to run.

Begin by confirming scope with the user: full pipeline (all three stages) or starting from a later
stage if artifacts already exist. Then invoke Stage 1.
