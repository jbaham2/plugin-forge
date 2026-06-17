---
name: build-prompt
description: This skill should be used when the user asks to "build the plugin prompt", "fill in the plugin-creator template", "turn these docs into a builder prompt", "research this service for a plugin", or wants a ready-to-run plugin-builder prompt generated from docs (or a doc roadmap) — Stage 2 only, producing a prompt rather than the built plugin. This is Stage 2 of the plugin-forge pipeline. If the user gives a bare docs URL and wants an end-to-end docs→plugin run (not just the builder prompt), use `forge` instead.
argument-hint: <docs-url> [topic-slug]
allowed-tools: WebFetch, WebSearch, Read, Write
version: 0.1.0
---

# Build Prompt — research-and-fill the plugin-builder template

Given a documentation URL (and optionally a doc roadmap from Stage 1), research the service and
produce a ready-to-run plugin-builder prompt by filling the template in
`references/plugin-creator-template.md`. Save it to `<topic>-plugin-prompt.md`. The output is the
prompt Stage 3 (`build-plugin`) executes.

## Inputs

- `<docs-url>` (required) — the documentation link or website to analyze.
- `[topic-slug]` (optional) — kebab-case slug for the filename. If omitted, derive it from the
  service's canonical name (e.g. "PostHog" → `posthog`). Output: `<topic>-plugin-prompt.md`.

## Operating principles

- **Ground every claim in a source.** Assert only capabilities/workflows/gotchas actually read on
  a fetched page or search result. Never invent features. Mark uncertain items `[UNVERIFIED]`.
- **Distill judgment, don't copy facts.** When seeding the source tracker, capture *what each
  source teaches* (a workflow, decision, failure mode) — not API signatures or code, which the
  downstream build fetches live.
- **Fill, don't rewrite.** Preserve the template's structure, principles, guardrails, and
  definition-of-done verbatim. Only resolve `{{...}}` placeholders and append the pre-seeded source
  tracker + research summary.

## Research protocol

0. **Use an existing roadmap if one exists.** Before crawling, check the working directory for a
   `<topic>-doc-roadmap.md` (Stage 1 output), or use one the user points to. If found, READ IT and
   treat it as the authoritative docs map: its flat index seeds the source tracker, its `proposed
   pillar` column seeds `{{DOMAINS}}`, and its category counts indicate where depth lives. In that
   case SKIP the broad crawl in steps 1–2 and spot-fetch only the few pages needed to confirm
   `{{SERVICE}}`, `{{SCOPE}}`, and `{{AUDIENCE}}`, then go to steps 3–5. If no roadmap exists, do
   full discovery in steps 1–4.
1. **Read the entry point.** Fetch `<docs-url>`. Identify the canonical product name, a one-sentence
   description, the audience, and the top-level docs structure.
2. **Map the docs surface.** Fetch key sub-pages — quickstart, core concepts, main feature areas,
   API/SDK reference index, self-hosting/deployment, pricing/editions. Prefer clean-markdown
   (`.md`-append, `llms.txt`). Note languages/runtimes/editions (→ `{{SCOPE}}`).
3. **Discover existing assets.** Search for an official/high-quality Claude skill, plugin, docs-MCP,
   `llms.txt`, or CLI (queries like `"<service> claude skill"`, `"<service> mcp server"`,
   `"<service> llms.txt"`, `"<service> cli"`). This drives the vendor-vs-build decision downstream.
4. **Search for workflows & case studies.** Find tutorials, guides, academy/courses, blog deep-dives
   (→ playbooks) and real-world walkthroughs, cookbooks, example repos, "how X uses <service>"
   write-ups, postmortems (→ failure taxonomies, gotchas, recipes). Capture title + URL + the one
   durable thing each teaches.
5. **Derive the placeholders** (see below).

## Deriving the placeholders

- `{{SERVICE}}` — the canonical product name exactly as the vendor writes it.
- `{{DOMAINS}}` — 3–6 expertise pillars matching how the docs are organized, ordered
  highest-value-first.
- `{{SCOPE}}` — languages/runtimes/editions in scope (e.g. "Python + TypeScript; Cloud + self-host").
- `{{AUDIENCE}}` — the primary user the docs target.
- `{{topic}}` — kebab-case slug of `{{SERVICE}}` (unless supplied).

For any placeholder the research can't settle confidently, fill the best inference and flag it
inline as `{{… }}  ⚠ inferred — confirm`.

## Output — write `<topic>-plugin-prompt.md`

1. **Header** — `# <Service> → Expert Plugin Builder Prompt` plus a 2–3 line summary.
2. **Research summary** (for the human) — the one-sentence definition; chosen `{{DOMAINS}}` with a
   one-line rationale each; the `{{SCOPE}}` and `{{AUDIENCE}}`; and the vendor-vs-build verdict
   (official asset found, or "build from scratch").
3. **The complete PROMPT block from `references/plugin-creator-template.md`**, copied verbatim with
   every `{{…}}` resolved. Do not alter its sections, principles, guardrails, phased plan,
   per-skill checklist, or definition of done.
4. **Pre-seeded source tracker** appended at the end of the PROMPT block (markdown table):
   `title | url | type (doc/literature/case-study) | priority (P1→P3) | target pillar | what it teaches | status (todo)`.
   Include the existing-asset finding as its own row if one was found.

Keep sections 3+4 runnable as-is — pasteable into an empty repo with no further discovery needed.

## Definition of done

- If a `<topic>-doc-roadmap.md` existed, it was read and used (no redundant full crawl); otherwise
  `<docs-url>` and key sub-pages were actually fetched. Claims trace to sources.
- All five placeholders resolved (inferred ones flagged).
- Existing official asset was searched for and the verdict recorded.
- `<topic>-plugin-prompt.md` contains header, research summary, the verbatim filled PROMPT block,
  and a pre-seeded source tracker with ≥1 case-study/workflow row.
- The embedded PROMPT block is byte-for-byte the template except for placeholder values.

## Additional resources

- **`references/plugin-creator-template.md`** — the template whose PROMPT block is filled and
  embedded verbatim. Read it before producing the output.
- **`references/methodology.md`** — background on how the reference plugin (Langfuse) was built;
  consult for the distill-judgment/fetch-facts rationale.

Begin with research protocol Step 0 (check for a roadmap), then proceed; report findings at each
step before writing the file.
