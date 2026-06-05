# gfargo/skills

A Claude Code plugin **marketplace** for [Griffen Fargo](https://github.com/gfargo)'s published skills. Add it once and install whatever you want — new skills land here over time.

## Install

```bash
# In Claude Code — add the marketplace (one time)
/plugin marketplace add gfargo/skills

# Install a plugin (its skills become available, namespaced by plugin)
/plugin install terminal@gfargo-skills
```

Update later with `/plugin marketplace update gfargo-skills`.

## Plugins

### `terminal` — terminal tooling

| Skill | What it does |
|-------|--------------|
| `terminal:tui-design` | Design and build clean, professional, minimal TUIs and CLIs (Bubble Tea / Ratatui / Textual / Ink), with library selection, layout, and interaction patterns. |
| `terminal:vhs-cli-demos` | Generate deterministic screenshots and demo GIFs of CLI/TUI apps with [Charm VHS](https://github.com/charmbracelet/vhs) — tape authoring, determinism, motion-GIF storytelling, and lossless size optimization. |

These two pair naturally: design a terminal app, then capture it for your README, docs, or marketing site.

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json        # the catalog (lists every plugin)
└── plugins/
    └── terminal/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            ├── tui-design/      # SKILL.md + references/
            └── vhs-cli-demos/   # SKILL.md + references/ + scripts/
```

Adding a new skill = drop a `skills/<name>/` folder into the relevant plugin (or add a new plugin under `plugins/` and list it in `marketplace.json`), then bump the version.

## License

MIT
