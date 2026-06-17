---
name: build-plugin
description: This skill should be used when the user asks to "run the plugin-prompt", "execute the builder prompt", "create the expert plugin from this prompt", or wants the actual Claude Code plugin built from an already-filled <topic>-plugin-prompt.md (Stage 2 output MUST exist). This is Stage 3 of the plugin-forge pipeline. Precondition: a filled <topic>-plugin-prompt.md is present. If the user only has docs and no filled prompt, use `forge` (full pipeline) or `build-prompt` (Stage 2) instead.
argument-hint: [path-to-plugin-prompt.md]
allowed-tools: Read, Write, Edit, Bash, WebFetch, WebSearch, Skill, Glob, Grep, Agent
version: 0.1.0
---

# Build Plugin — execute the filled builder prompt

Take a filled `<topic>-plugin-prompt.md` (Stage 2 output) and build the target Claude Code expert
plugin by executing its embedded PROMPT block — the plugin-creator build protocol.

## Inputs

- `[path-to-plugin-prompt.md]` (optional) — path to the filled prompt. If omitted, look for a
  `*-plugin-prompt.md` in the working directory; if several exist, ask which to use.

## Where to build — critical

The plugin-creator protocol assumes a **fresh, empty git repository**. Before building:

1. Read the chosen `<topic>-plugin-prompt.md` and identify `{{SERVICE}}`/`{{topic}}`.
2. Determine a target directory for the new plugin (default: `./<topic>-plugin`, a sibling to the
   prompt file). Confirm the location with the user.
3. If the target directory does not exist, create it and `git init`. Do NOT build into a directory
   that already contains an unrelated project — the protocol writes scaffolding (`.claude-plugin/`,
   `skills/`, `workbench/`, `meta/`) and expects a clean slate.

## How to execute

1. **Load the build prompt as instructions.** Read the embedded PROMPT block from the chosen file
   (the section between "PROMPT (copy from here)" and "(end of PROMPT)"). Treat that block as the
   governing instructions for the build — including its `<operating_principles>`, `<phased_plan>`,
   `<per_skill_checklist>`, `<guardrails>`, and `<definition_of_done>`.
2. **Honor the start protocol.** The prompt begins by researching existing assets, proposing a
   skill ownership map, and asking up to 5 clarifying questions. Run that start protocol and WAIT
   for the user's answers before scaffolding. Present the Phase 0 plan for approval first.
3. **Reuse the pre-seeded source tracker.** The filled prompt already contains a source tracker
   from Stage 2 — use it as the starting tracker rather than rediscovering sources.
4. **Use platform builders/reviewers** as the prompt's `<tools_to_use>` directs: load
   `plugin-dev:skill-development` / `plugin-dev:plugin-structure` for authoring, the
   `plugin-dev:skill-reviewer` agent per skill, and the `plugin-dev:plugin-validator` agent before
   release. Record verdicts in the new plugin's `meta/DECISIONS.md`.
5. **Build YAGNI-style.** Author one skill fully end-to-end (author → review → validate) before
   scaffolding the next. Knowledge skills before tooling (commands/agents/hooks).

## Guardrails (from the build protocol)

- Distill durable judgment into `references/`; fetch stale-prone facts (APIs, code, versions, pricing)
  live — never embed them.
- Never duplicate a vendored asset; redraw the boundary instead.
- No empty scaffolding ahead of content. No `README.md` inside `commands/` or `agents/`. No comments
  in JSON. No hardcoded secrets — use `${ENV_VAR}`.
- Keep authoring scratch (`workbench/`) out of the shipped plugin via `.gitignore`.

## Definition of done

Defer to the `<definition_of_done>` inside the executed prompt. In summary: the plugin installs
cleanly with no load errors; every pillar has a reviewed, validated skill; any official asset is
vendored and attributed with no duplication; `references/` hold durable judgment while facts are
fetched live; commands/agents/hooks are thin, scoped, validated; README/LICENSE/`.gitignore` are
correct; and `meta/` trackers reflect the final state.

## Additional resources

These reference files live under the sibling `build-prompt` skill (not this skill's directory) — a
deliberate single-source-of-truth choice; preserve the relative path if refactoring layout:

- **`../build-prompt/references/plugin-creator-template.md`** — the canonical template, if the
  embedded block needs cross-checking against the original.
- **`../build-prompt/references/methodology.md`** — the build methodology this protocol distills.

Begin by locating and reading the filled prompt, confirming the target build directory, then run
the prompt's start protocol and wait for approval before scaffolding.
