# plugin-forge

Turn any service's documentation into a **Claude Code expert plugin**.

plugin-forge packages a three-stage pipeline that takes a single documentation URL and produces a
fully built, reviewed Claude Code plugin that makes Claude an expert at that service. What used to
be copy-pasted meta-prompts is now installable skills and slash commands.

- **Input:** a docs link — `https://docs.someservice.com/`
- **Output:** a working plugin — `./someservice-plugin/` with skills, references, and a manifest

---

## Table of contents

- [How it works](#how-it-works)
- [Installation](#installation)
- [Usage](#usage)
- [Worked example](#worked-example)
- [What each stage produces](#what-each-stage-produces)
- [Prerequisites](#prerequisites)
- [Design principles](#design-principles)
- [Components](#components)
- [Tips & troubleshooting](#tips--troubleshooting)
- [License](#license)

---

## How it works

```
map-docs      <docs-url>                 →  <topic>-doc-roadmap.md      (Stage 1)
build-prompt  <docs-url>                 →  <topic>-plugin-prompt.md    (Stage 2)
build-plugin  <topic>-plugin-prompt.md   →  ./<topic>-plugin/           (Stage 3)
forge         <docs-url>                 →  runs all three end-to-end
```

| Stage | Name | What it does | Output |
|-------|------|--------------|--------|
| 1 | `map-docs` | Interactively crawls the docs site into a complete coverage map — overviews, features, guides, examples, cookbooks, API reference, and in-page headers. | `<topic>-doc-roadmap.md` |
| 2 | `build-prompt` | Researches the service (reusing the roadmap if present), derives the builder placeholders, and fills the plugin-creator template. | `<topic>-plugin-prompt.md` |
| 3 | `build-plugin` | Executes the filled prompt's build protocol to scaffold, author, and review the actual plugin. | `./<topic>-plugin/` |
| — | `forge` | Orchestrates Stages 1 → 2 → 3 with confirmation checkpoints between them. | all of the above |

Each stage works two ways. The logic lives in a **skill** that Claude activates automatically when
your request matches its trigger phrases (e.g. "forge a plugin from https://docs.example.com" →
`forge`). A thin **command wrapper** also gives an explicit slash-command entry point,
`/plugin-forge:<stage>`, which invokes the matching skill with your arguments.

Stage 1 is recommended but optional: skip it and `build-prompt` self-crawls the docs (less
exhaustive). Each stage's output is the next stage's input, keyed by a shared `<topic>` slug so
filenames line up: `<topic>-doc-roadmap.md` → `<topic>-plugin-prompt.md` → `<topic>-plugin/`.

---

## Installation

### Option A — marketplace (recommended)

The repository root is a marketplace catalog (`.claude-plugin/marketplace.json`). From inside
Claude Code, point it at the GitHub remote:

```
/plugin marketplace add jbaham2/plugin-forge
/plugin install plugin-forge@plugin-forge-marketplace
```

Or add it from a local clone/checkout instead of the remote:

```
/plugin marketplace add /path/to/plugin-forge
/plugin install plugin-forge@plugin-forge-marketplace
```

Verify it loaded:

```
/help          # the /plugin-forge:* commands appear
/plugin        # plugin-forge is listed and enabled
```

### Option B — local directory (for development)

Point Claude Code straight at the plugin directory:

```bash
cc --plugin-dir /home/jbaham2/prompts/plugin-creator/plugin-forge
```

No restart is needed when you edit components — changes take effect in the next session.

---

## Usage

Run the whole pipeline by slash command:

```
/plugin-forge:forge https://docs.example.com/
```

…or just by intent — *"Forge a Claude Code plugin from https://docs.example.com/"*.

Run a single stage à la carte (slash command or natural language both work):

```
/plugin-forge:map-docs      https://docs.example.com/ example
/plugin-forge:build-prompt  https://docs.example.com/ example
/plugin-forge:build-plugin  example-plugin-prompt.md
```

**Arguments**

| Command | Arg 1 | Arg 2 |
|---------|-------|-------|
| `forge` | `<docs-url>` (required) | `[topic-slug]` (optional) |
| `map-docs` | `<docs-url>` (required) | `[topic-slug]` (optional) |
| `build-prompt` | `<docs-url>` (required) | `[topic-slug]` (optional) |
| `build-plugin` | `[path-to-plugin-prompt.md]` (optional) | — |

`topic-slug` is derived from the product name when omitted (e.g. "PostHog" → `posthog`).

> **Which command?** Give a bare docs URL and want the whole thing built → `forge`. Want only the
> coverage map → `map-docs`. Want only the filled builder prompt (to review/edit before building) →
> `build-prompt`. Already have a filled `*-plugin-prompt.md` → `build-plugin`.

---

## Worked example

Building an expert plugin for a hypothetical email API at `https://docs.example-mail.com/`:

**1. Forge it**

```
/plugin-forge:forge https://docs.example-mail.com/ example-mail
```

**2. Stage 1 — `map-docs` (interactive).** Claude reports what it found and checkpoints with you:

```
Checkpoint A — Scope
  Product: Example Mail        Docs root: https://docs.example-mail.com/
  Signals: sitemap.xml ✓, llms.txt ✓, OpenAPI spec ✓
  Proposed boundary: docs.example-mail.com/* (excluding /blog)
  Estimated pages: ~64
  → Confirm boundary & depth? (how deep into the API reference?)
```

You steer depth section by section; it writes `example-mail-doc-roadmap.md` — a tree + flat index
of every page tagged by category (overview / guide / how-to / example / cookbook / api / …) with
each page's H2/H3 headers, plus proposed expertise pillars.

**3. Stage 2 — `build-prompt`.** Detects the roadmap, skips re-crawling, derives placeholders, and
writes `example-mail-plugin-prompt.md`:

```
Research summary
  SERVICE  = Example Mail
  DOMAINS  = setup, sending, templates, webhooks, deliverability   (highest-value first)
  SCOPE    = Node + Python SDKs; REST API
  AUDIENCE = backend engineers integrating transactional email
  Vendor verdict: no official Claude skill found → build from scratch
```

…followed by the verbatim builder prompt with those values filled in and a pre-seeded source
tracker (every roadmap page as a row). Review it, fix any `⚠ inferred — confirm` items.

**4. Stage 3 — `build-plugin`.** Confirms a target directory (`./example-mail-plugin/`), runs the
build prompt's start protocol (asks a few clarifying questions), then scaffolds and authors one
skill per pillar — reviewing and validating each before moving on.

**Result:** `./example-mail-plugin/` — an installable plugin whose skills make Claude an expert at
Example Mail.

> The service name and URL above are illustrative. Real runs ground every claim in pages actually
> fetched and flag anything unverified.

---

## What each stage produces

- **`<topic>-doc-roadmap.md`** — a complete coverage map: a hierarchical tree mirroring the site,
  a flat index table (`title | url | category | priority | pillar | notes`), per-page headers, and
  a coverage summary. This is the exhaustive inventory that keeps the build from missing topics.
- **`<topic>-plugin-prompt.md`** — a human-readable research summary plus a ready-to-run builder
  prompt (the plugin-creator template with placeholders resolved) and a pre-seeded source tracker.
  Self-contained: pasteable into a fresh repo even without the rest of plugin-forge.
- **`./<topic>-plugin/`** — the built plugin: `.claude-plugin/plugin.json`, one skill per expertise
  pillar with distilled `references/`, any vendored official asset, README, LICENSE, and `meta/`
  trackers.

---

## Prerequisites

- **Network access** — Stages 1–2 use WebFetch/WebSearch to read documentation.
- **plugin-dev** (recommended) — Stage 3 uses the `plugin-dev` skills/agents
  (`skill-development`, `plugin-structure`, `skill-reviewer`, `plugin-validator`) to author and
  validate the plugin it builds. Without them, Stage 3 still works but hand-rolls those steps.
- **An empty target directory** for Stage 3 — the build protocol expects a clean git repo and
  writes scaffolding (`.claude-plugin/`, `skills/`, `workbench/`, `meta/`) into it.

---

## Design principles

plugin-forge bakes in the approach that produced the reference build (see
`skills/build-prompt/references/methodology.md`):

- **Distill judgment, fetch facts.** Durable judgment (workflow sequencing, "which approach when"
  decision tables, failure taxonomies, checklists) goes into the built plugin's `references/`.
  Stale-prone facts (API signatures, code, version numbers, pricing) stay OUT and are fetched live.
  This keeps generated plugins from rotting as the service changes.
- **Vendor + complement, never duplicate.** Stage 2 searches for an official skill/plugin/MCP/CLI;
  if one exists, the build vendors it and authors only the gaps.
- **Progressive disclosure & YAGNI.** Lean skills with depth in `references/`; one skill built
  end-to-end (author → review → validate) before the next.
- **Ground every claim.** Research asserts only what was actually read; unverified items are flagged
  `[UNVERIFIED]` and inferred placeholders get `⚠ inferred — confirm`.

---

## Components

Skills + thin command wrappers (no hooks, agents, or MCP):

```
plugin-forge/
├── .claude-plugin/plugin.json
├── LICENSE
├── README.md
├── commands/                   # thin slash-command wrappers → invoke the matching skill
│   ├── forge.md
│   ├── map-docs.md
│   ├── build-prompt.md
│   └── build-plugin.md
└── skills/
    ├── forge/SKILL.md          # orchestrator
    ├── map-docs/SKILL.md       # Stage 1
    ├── build-prompt/SKILL.md   # Stage 2
    │   └── references/
    │       ├── plugin-creator-template.md   # the verbatim builder template (filled by Stage 2)
    │       └── methodology.md               # build-methodology background
    └── build-plugin/SKILL.md   # Stage 3
```

`plugin-creator-template.md` is the prompt Stage 2 fills and Stage 3 executes. `build-plugin`
references both files via `../build-prompt/references/` — a deliberate single-source-of-truth choice.

---

## Tips & troubleshooting

- **Always run Stage 1 for large docs.** The roadmap is what makes coverage exhaustive; skipping it
  trades thoroughness for speed.
- **Review the filled prompt before building.** Stage 2's output is the cheapest place to correct a
  wrong pillar list or scope — edit `<topic>-plugin-prompt.md` before running Stage 3.
- **A stage double-triggers?** Command and skill share trigger surfaces by design (command =
  explicit, skill = by intent); the wrappers only delegate, so there's no conflicting logic.
- **JS-heavy docs return little.** If WebFetch can't read a rendered-only site, Stage 1 will say so
  rather than invent pages — supply an `llms.txt`/sitemap URL or specific sub-pages.
- **Generated artifacts** (`*-doc-roadmap.md`, `*-plugin-prompt.md`) are git-ignored by this
  plugin's own `.gitignore` so they don't clutter the repo.

---

## License

MIT — see [LICENSE](./LICENSE).
