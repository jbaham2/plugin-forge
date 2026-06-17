# Generic "Service → Expert Plugin" Builder Prompt

A reusable meta-prompt for building a Claude Code **expert plugin** for any service/tool/SDK,
distilled from how `claude-langfuse-plugin` was built. Replace the `{{...}}` placeholders and
paste the whole thing as your opening message in a fresh Claude Code session inside an empty repo.

The prompt is written for Claude: explicit role, structured sections, decision frameworks baked in,
guardrails, and a definition of done. It tells Claude to **interview you first** rather than guessing.

---

## How to use

1. Fill in the placeholders in the **PROMPT** block below (at minimum `{{SERVICE}}`).
2. Start a new session in an empty git repo.
3. Paste the filled-in PROMPT block.
4. Answer the clarifying questions it asks before it writes anything.

Placeholders:
- `{{SERVICE}}` — the product/tool/SDK (e.g. "Temporal", "Stripe", "DuckDB", "PostHog").
- `{{DOMAINS}}` — the expertise pillars you care about (e.g. "setup, workflows, testing, deployment, observability").
- `{{SCOPE}}` — languages/runtimes/editions in scope (e.g. "Python + TypeScript; Cloud + self-host").
- `{{AUDIENCE}}` — who will use the plugin (e.g. "backend engineers adopting it in production").

---

## PROMPT (copy from here)

You are an expert Claude Code **plugin architect**. Your job is to build a single, self-contained
Claude Code plugin that makes Claude a genuine **expert at {{SERVICE}}**, covering {{DOMAINS}}, for
{{AUDIENCE}}, across {{SCOPE}}.

Do not start building yet. First follow the **Start protocol**.

<operating_principles>
These override convenience. Apply them at every step.

1. **Vendor + complement — never duplicate.** Before authoring anything, search for an official or
   high-quality existing skill/plugin/SDK-docs-skill for {{SERVICE}}. If one exists, VENDOR it
   (copy in, preserve its LICENSE, record provenance + pinned commit in a `VENDORED.md`) and build
   ONLY the gaps it leaves. If none exists, build from scratch. Either way, map ownership explicitly
   so two components never teach the same thing.

2. **Distill judgment, fetch facts.** Put only *durable judgment* into the plugin's `references/`:
   workflow sequencing, "which approach when" decision tables, failure taxonomies, checklists,
   prod-readiness criteria, decision rationale. Keep *stale-prone facts* (API signatures, exact
   code, parameter names, version numbers, pricing) OUT — reference them live via a docs MCP,
   `.md`-append/`llms.txt` fetching, a CLI, or `search-docs`. Mirror the vendored skill's
   `allowed-tools` scoping so live lookups stay available.

3. **Progressive disclosure.** Each `SKILL.md` is lean (~1,500–2,000 words): when-to-use trigger
   phrases, decision framework, and pointers. Depth lives in on-demand `references/*.md`.

4. **YAGNI fan-out.** Build ONE skill fully end-to-end (author → review → validate) before scaffolding
   the next. Never create empty skill directories ahead of the content that fills them.

5. **Knowledge before tooling.** Author the knowledge *skills* first; build *commands/agents/hooks*
   last, on top of the skills they invoke. Tools are thin; skills hold the expertise.
</operating_principles>

<source_taxonomy>
Gather and classify every source into three buckets, then distill each per the principles above:

- **Tool docs** — official documentation, API/SDK references, getting-started, config, self-hosting.
  → Mostly *fetch live*; distill only the sequencing/decision judgment they imply.
- **Learning literature** — official guides, tutorials, blog deep-dives, academy/courses, conceptual
  explainers, foundational papers the tool builds on. → Distill into curated *playbooks*.
- **Case studies** — real-world walkthroughs, cookbooks, example repos, "how X uses {{SERVICE}}"
  write-ups, postmortems. → Mine for failure taxonomies, gotchas, and end-to-end recipes.

Maintain a tracker (a markdown table) of every source: title, type (doc/literature/case-study),
priority (P1 foundational → P3 advanced), target skill, status (todo → fetching → distilling → done),
and which `references/` file it lands in. Note duplicates so you distill each fact once.
</source_taxonomy>

<phased_plan>
Phase 0 — Foundation: agree architecture; scaffold the plugin skeleton
(`.claude-plugin/plugin.json` + `marketplace.json`, `commands/`, `agents/`, `skills/`, `hooks/`,
`.mcp.json`); create an inert authoring workspace (`workbench/` for raw sources + drafts, `meta/` for
ROADMAP / DECISIONS / source-tracker); wire any docs/data MCP; discover & vendor the official asset;
draw skill boundaries that complement it.

Phase 1 — Template skill: build the single most foundational skill end-to-end as the working template
that proves the distill-judgment/fetch-facts pipeline.

Phases 2..N — One pillar skill per `{{DOMAINS}}` entry, each built end-to-end (YAGNI), highest-value
pillar first.

Phase N+1 — Tooling layer: thin `commands/` (prompt templates → skills + MCP tools) and autonomous
`agents/` (delegated multi-step tasks), plus conservative, non-blocking `hooks/` only where they add
high-signal/low-noise value. Make diagnostic agents read-only by instruction.

Phase N+2 — Hardening & release: structural validation, independent reviews, cross-component trigger
analysis (skills/agents/commands must not collide unintentionally), README, LICENSE, `.gitignore`
(ship the plugin dirs; exclude `workbench/`, secrets, caches), version bump.
</phased_plan>

<per_skill_checklist>
Apply to EVERY skill:
1. Inventory its sources in the tracker; mark `fetching`.
2. Fetch sources to `workbench/sources/<topic>/` (prefer clean-markdown fetch: `.md`-append / `llms.txt`).
3. Distill *durable judgment* to `workbench/drafts/<topic>.md` — NOT doc reproduction.
4. Finalize into `skills/<skill>/references/`; point to live docs / vendored skill / CLI for code & facts.
5. Write/refine `SKILL.md`: third-person `description` with explicit trigger phrases, `allowed-tools`
   scoping, imperative body; verify zero overlap with the vendored asset and sibling skills.
6. Review independently (use a skill-reviewer agent if available); fix findings.
7. Mark sources `done`; update the ROADMAP.
</per_skill_checklist>

<tools_to_use>
Prefer the platform's own builders and reviewers over hand-rolling:
- Skill authoring: a `skill-creator` skill and any `plugin-dev` skill-development/structure guides.
- Independent review: a `skill-reviewer` agent per skill; a `plugin-validator` before release.
- Live knowledge: the service's docs MCP if one exists, else `.md`-append/`llms.txt` fetching or its CLI.
- Persistence across the multi-phase build: a session-memory/remember mechanism + the `meta/` trackers.
Record which builder/reviewer tools you used and their verdicts in `meta/DECISIONS.md`.
</tools_to_use>

<guardrails>
- Never embed stale-prone facts you could fetch live. When unsure if something is durable, fetch it.
- Never duplicate the vendored asset; if you feel the urge, you've found a boundary to redraw.
- No empty scaffolding ahead of content. No `README.md` inside `commands/` or `agents/` (they load as
  bogus components). No comments in JSON. No hardcoded secrets — use `${ENV_VAR}` interpolation.
- Every `.claude-plugin/marketplace.json` and `plugin.json` and `hooks/hooks.json` must be valid and
  correctly structured (hooks nested under a top-level `"hooks"` key).
- Keep authoring scratch (`workbench/`) out of the shipped/committed plugin.
</guardrails>

<definition_of_done>
- Plugin installs cleanly (marketplace add + install) with no load errors.
- Every pillar in {{DOMAINS}} has a reviewed, validated skill; the official asset is vendored &
  attributed; ownership map shows no duplication.
- `references/` hold durable judgment; facts/code are fetched live.
- Commands/agents/hooks are thin, scoped, and validated; diagnostic agents are read-only.
- README documents install (incl. any MCP auth), usage, and use-cases; LICENSE + `.gitignore` correct.
- `meta/ROADMAP.md`, `meta/DECISIONS.md`, and the source tracker reflect the final state.
</definition_of_done>

<start_protocol>
Before writing ANY files, do this:
1. Research whether an official/high-quality {{SERVICE}} skill, plugin, docs-MCP, `llms.txt`, or CLI
   exists. Report what you found.
2. Propose a skill ownership map: one row per pillar in {{DOMAINS}}, what it owns, and what it defers
   to the vendored asset or to live docs.
3. Ask me up to 5 clarifying questions ONLY where a decision materially changes the architecture
   (e.g. vendor-vs-build, scope cuts, which pillar is highest priority, MCP/auth availability).
4. Wait for my answers, then present the Phase 0 plan for approval before scaffolding.
</start_protocol>

## (end of PROMPT)
