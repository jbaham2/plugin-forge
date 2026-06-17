# Build Methodology — how the reference expert plugin was built

Background for the distill-judgment/fetch-facts approach the builder prompt is based on, reconstructed
from the reference build (`claude-langfuse-plugin`). Use it to understand *why* the build protocol is
shaped the way it is.

## Initial intent

Build a Claude Code plugin that makes Claude an expert at a service — for the reference build,
Langfuse — across pillars such as setup, observability, prompt management, evaluation, and monitoring.

## Key decisions that shaped the build

1. **Repo shape.** One plugin repo with knowledge co-located in skills, rather than a separate
   "knowledge brain." The inert authoring workspace (`workbench/`, `meta/`) lives in the same repo but
   ships excluded.
2. **Discovery of an official asset.** The service shipped its own MIT-licensed skill. The response was
   to **vendor + complement** it — copy it in, preserve its LICENSE, record provenance — rather than
   rebuild from scratch or depend on a second install.
3. **Knowledge philosophy — distill judgment, fetch facts.** Durable judgment (sequencing,
   "which method when," checklists, failure taxonomies) goes into `references/`; stale-prone facts
   (API signatures, code, versions, pricing) stay live via a docs-MCP / `.md`-append fetching.
4. **Boundary reshaping.** Skills were deleted or redrawn so nothing duplicated the vendored asset
   (e.g. dropping an observability skill, renaming core→setup) — duplication is a signal to redraw a
   boundary.
5. **Scope.** Cloud + self-host; Python + JS/TS + frameworks; MCP wired via `.mcp.json`.

## Tools and agents used

- **skill-creator** — authored each `SKILL.md`.
- **plugin-dev skills** (`skill-development`, `plugin-structure`, `command-development`) — guidance.
- **plugin-dev:skill-reviewer** agent — independent content review per skill (pass + fixes each time).
- **plugin-dev:plugin-validator** agent — final structural pass (caught a `.mcp.json` comment-key bug).
- **docs MCP + `.md`-append fetching** — live source retrieval.
- **a remember/session-memory mechanism + `meta/` trackers** — persistent state across the
  multi-phase build.

## The repeatable pipeline

```
scrape → workbench/sources/ → distill judgment → workbench/drafts/
       → finalize into skills/*/references/ → SKILL.md via skill-creator
       → skill-reviewer → validate
```

Tracked phase by phase, governed by a YAGNI rule: build one skill end-to-end (author → review →
validate) before starting the next, and author knowledge skills before any tooling.
