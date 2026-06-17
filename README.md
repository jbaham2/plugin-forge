# plugin-forge marketplace

A Claude Code [plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins) hosting
**plugin-forge** — a plugin that turns any service's documentation into a Claude Code expert plugin.

## What's here

This repository is a marketplace catalog. The plugin itself lives in
[`plugin-forge/`](./plugin-forge) — see its [README](./plugin-forge/README.md) for full docs.

```
.
├── .claude-plugin/
│   └── marketplace.json     # the marketplace catalog (source → ./plugin-forge)
└── plugin-forge/            # the plugin (manifest, commands, skills)
    └── README.md            # full plugin documentation
```

## Install

From inside Claude Code:

```
/plugin marketplace add jbaham2/plugin-forge
/plugin install plugin-forge@plugin-forge-marketplace
```

## What plugin-forge does

Give it a documentation URL and it builds a complete expert plugin for that service through a
three-stage pipeline:

```
map-docs      <docs-url>                 →  <topic>-doc-roadmap.md      (Stage 1: map the docs)
build-prompt  <docs-url>                 →  <topic>-plugin-prompt.md    (Stage 2: fill the builder prompt)
build-plugin  <topic>-plugin-prompt.md   →  ./<topic>-plugin/           (Stage 3: build the plugin)
forge         <docs-url>                 →  runs all three end-to-end
```

Full usage, a worked example, and design notes are in the
[plugin README](./plugin-forge/README.md).

## License

MIT — see [plugin-forge/LICENSE](./plugin-forge/LICENSE).
