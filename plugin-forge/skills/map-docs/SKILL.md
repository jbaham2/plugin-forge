---
name: map-docs
description: This skill should be used when the user asks to "map the docs", "build a doc roadmap", "crawl this documentation site", "create a coverage map of these docs", or wants an exhaustive inventory/coverage map of a documentation website (overviews, features, guides, examples, cookbooks, API reference, and in-page headers) — Stage 1 only. This is Stage 1 of the plugin-forge pipeline. If the user gives a bare docs URL and wants an end-to-end docs→plugin run (not a roadmap specifically), use `forge` instead.
argument-hint: <docs-url> [topic-slug]
allowed-tools: WebFetch, WebSearch, Read, Write, AskUserQuestion
version: 0.1.0
---

# Map Docs — Documentation Roadmap Builder

Work interactively with the user to traverse a documentation website and produce a complete,
hierarchical **coverage map** — from the how-to/getting-started side through the full API
reference — capturing pages AND their in-page headers (H1→H3) so no concept is missed. The
output feeds Stage 2 (`build-prompt`).

## Inputs

- `<docs-url>` (required) — the documentation root or website to map.
- `[topic-slug]` (optional) — kebab-case slug for the filename. If omitted, derive it from the
  product name (e.g. "PostHog" → `posthog`). Output file: `<topic>-doc-roadmap.md`.

## Operating principles

- **Completeness over speed.** The goal is full coverage, not a summary. Every nav entry, sidebar
  section, and API resource is a node to capture. Breadth first, then depth.
- **Map what exists, never invent.** Record only pages and headers discovered via a fetched page,
  sitemap, or nav. Mark implied-but-unconfirmed sections `[unconfirmed]` and ask — never fabricate
  titles or URLs.
- **Collaborate, don't guess.** Pause at each checkpoint, surface gaps and ambiguities, and wait
  for direction before continuing. The user may redirect depth (go deep here, skip there).
- **Headers are first-class.** Capture each page's H1→H3 structure; sub-headers catch concepts the
  nav labels don't reveal.

## Discovery strategy (in order)

1. **Machine-readable maps first** — try `<docs-url>/sitemap.xml`, `/llms.txt`, `/llms-full.txt`,
   and `.md`-append / `/docs.md` variants. These often enumerate every page cheaply.
2. **Navigation tree** — fetch the root, parse the primary nav/sidebar and any "all docs" index;
   record the hierarchy exactly as the site organizes it.
3. **API reference** — locate it separately (often a distinct sub-site or OpenAPI/Swagger spec) and
   capture every resource/endpoint group, not just the overview.
4. **Cross-links & "next" trails** — follow in-page next/related links to catch pages the nav omits
   (cookbooks, examples, migration guides, changelogs).

Stay within the docs domain/subpath unless the user approves going wider. Note anything behind auth.

## Page taxonomy

Classify every page into one primary category: **overview**, **feature**, **guide**,
**how-to/tutorial**, **example**, **cookbook**, **api**, **config/setup**, **reference-other**
(changelog, glossary, FAQ, limits, pricing, security, migration), or **uncategorized** (flag for
the user to label). For each page, also record its H2/H3 headers as a nested list.

## Interactive protocol — pause for confirmation at each checkpoint

- **Checkpoint A — Scope.** Report product name, docs root, available discovery signals
  (sitemap? llms.txt? OpenAPI?), the proposed crawl boundary, and an estimated page count. Use
  AskUserQuestion to confirm boundary and depth (how deep into the API, include changelog/blog?).
- **Checkpoint B — Top-level skeleton.** Present the top-level section tree, categorized. Ask which
  areas are high-priority and which to map shallowly or skip.
- **Checkpoints C…N — Section sweeps.** Map one top-level section at a time to full depth (pages +
  headers). After each, show the sub-tree, list `[unconfirmed]` or ambiguous nodes, and confirm
  before moving on. Keep a running tally of remaining sections. Never silently truncate — if a
  section is huge, say so and propose how to sample it.
- **Final checkpoint — Coverage review.** Show coverage stats per category, list known gaps and
  anything skipped by the user's direction, and confirm before writing the file.

## Output — write `<topic>-doc-roadmap.md`

On final confirmation, write the file with these sections:

1. **Header** — `# <Service> — Documentation Roadmap`, the docs root URL, the date, and a one-line
   product description.
2. **Coverage summary** — a table of page counts per category, plus explicitly-skipped areas (with
   reason) and any `[unconfirmed]` nodes.
3. **Roadmap tree** — the full hierarchy as a nested markdown list mirroring the site's own
   organization. Each leaf: title, URL, `[category]` tag, and an indented sub-list of its H2/H3
   headers.
4. **Flat index table** — every page as a row:
   `title | url | category | priority (P1→P3) | proposed pillar | notes`. The `proposed pillar`
   column groups pages into candidate expertise pillars (these become `{{DOMAINS}}` downstream).
5. **Handoff note** — tell Stage 2 (`build-prompt`) how to use this: the flat index seeds the
   source tracker, the proposed pillars seed `{{DOMAINS}}`, and category counts indicate where
   depth lives.

Keep URLs exact and complete so the downstream builder can fetch them directly.

## Definition of done

- Discovery used machine-readable maps where they existed, not just nav.
- Both the how-to side AND the full API reference are mapped, plus overviews, features, guides,
  examples, and cookbooks.
- Every captured page has a category tag and its H2/H3 headers.
- Every checkpoint was confirmed by the user before proceeding.
- `<topic>-doc-roadmap.md` contains all five sections; no invented pages; unconfirmed nodes flagged.

Begin at **Checkpoint A**: fetch `<docs-url>`, report scope and discovery signals, and confirm the
crawl boundary before mapping anything.
