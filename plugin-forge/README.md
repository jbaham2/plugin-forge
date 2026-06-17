# plugin-forge

Turn any service's documentation into a **Claude Code expert plugin** — as guided slash commands.

plugin-forge packages a three-stage pipeline that takes a documentation URL and produces a fully
built expert plugin for that service. What used to be copy-pasted meta-prompts is now installable
skills.

## The pipeline

```
map-docs      <docs-url>                 →  <topic>-doc-roadmap.md      (Stage 1)
build-prompt  <docs-url>                 →  <topic>-plugin-prompt.md    (Stage 2)
build-plugin  <topic>-plugin-prompt.md   →  ./<topic>-plugin/           (Stage 3)
forge         <docs-url>                 →  runs all three end-to-end
```

Each stage works two ways. The logic lives in a **skill** that Claude activates automatically when
your request matches its trigger phrases (e.g. "forge a plugin from https://docs.example.com" →
`forge`). A thin **command wrapper** also provides an explicit slash-command entry point,
`/plugin-forge:<stage>` (e.g. `/plugin-forge:forge <docs-url>`), which simply invokes the matching
skill with your arguments.

| Stage | Skill | What it does | Output |
|-------|-------|--------------|--------|
| 1 | `map-docs` | Interactively crawls the docs site into a complete coverage map (overviews, features, guides, examples, cookbooks, API reference, and in-page headers). | `<topic>-doc-roadmap.md` |
| 2 | `build-prompt` | Researches the service (reusing the roadmap if present), derives the builder placeholders, and fills the plugin-creator template. | `<topic>-plugin-prompt.md` |
| 3 | `build-plugin` | Executes the filled prompt's build protocol to scaffold and author the actual plugin. | `./<topic>-plugin/` |
| — | `forge` | Orchestrates Stages 1→2→3 with confirmation checkpoints between them. | all of the above |

Stage 1 is recommended but optional: if skipped, `build-prompt` self-crawls the docs (less
exhaustive). Each stage's output is the next stage's input, keyed by the shared `<topic>` slug.

## Installation

Test locally with:

```bash
cc --plugin-dir /path/to/plugin-forge
```

Or add it to a marketplace and install normally.

## Usage

Run the whole pipeline by slash command:

```
/plugin-forge:forge https://docs.example.com/
```

Or by intent — "Forge a Claude Code plugin from https://docs.example.com/".

Single stages à la carte (slash command or natural language both work):

```
/plugin-forge:map-docs https://docs.example.com/ example
/plugin-forge:build-prompt https://docs.example.com/ example
/plugin-forge:build-plugin example-plugin-prompt.md
```

## Prerequisites

- **Network access** — Stages 1–2 use WebFetch/WebSearch to read documentation.
- **plugin-dev** (recommended) — Stage 3 uses the `plugin-dev` skills/agents
  (`skill-development`, `plugin-structure`, `skill-reviewer`, `plugin-validator`) to author and
  validate the plugin it builds. Without them, Stage 3 still works but hand-rolls those steps.
- **An empty target directory** for Stage 3 — the build protocol expects a clean repo.

## Components

Skills + thin command wrappers (no hooks, agents, or MCP):

```
plugin-forge/
├── .claude-plugin/plugin.json
├── LICENSE
├── README.md
├── commands/                   # thin slash-command wrappers → invoke matching skill
│   ├── forge.md
│   ├── map-docs.md
│   ├── build-prompt.md
│   └── build-plugin.md
└── skills/
    ├── forge/SKILL.md          # orchestrator
    ├── map-docs/SKILL.md       # Stage 1
    ├── build-prompt/SKILL.md   # Stage 2
    │   └── references/
    │       ├── plugin-creator-template.md   # the verbatim builder template
    │       └── methodology.md               # build methodology background
    └── build-plugin/SKILL.md   # Stage 3
```

The plugin can be installed via the marketplace catalog one directory up
(`../.claude-plugin/marketplace.json`).

The `plugin-creator-template.md` is the prompt Stage 2 fills and Stage 3 executes; `methodology.md`
records the distill-judgment/fetch-facts approach the build protocol is based on.
