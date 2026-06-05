# gfargo/skills

> My personal **Claude Code skills marketplace**. Add it once, then install whichever skills you want — I add more over time.

[![skills.sh](https://skills.sh/b/gfargo/skills)](https://skills.sh/gfargo/skills)
[![Release](https://img.shields.io/github/v/release/gfargo/skills?label=release&color=2da44e)](https://github.com/gfargo/skills/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

I'm [Griffen Fargo](https://github.com/gfargo). This is where I publish the Claude Code skills I build — terminal tooling for now, more over time. Everything here follows the [Agent Skills](https://agentskills.io) standard and the [Claude Code plugin-marketplace](https://code.claude.com/docs/en/plugin-marketplaces) format, so it installs through Claude Code or [`npx skills`](https://github.com/vercel-labs/skills).

---

## Contents

- [Install](#install)
  - [In Claude Code](#in-claude-code)
  - [With `npx skills`](#with-npx-skills)
- [Skills](#skills)
  - [`terminal:tui-design`](#terminaltui-design)
  - [`terminal:vhs-cli-demos`](#terminalvhs-cli-demos)
- [Requirements](#requirements)
- [Structure](#structure)
- [Managing the marketplace](#managing-the-marketplace)
- [Adding a skill](#adding-a-skill)
- [References](#references)
- [License](#license)

---

## Install

### In Claude Code

```bash
/plugin marketplace add gfargo/skills
/plugin install terminal@gfargo-skills
```

Installs the `terminal` plugin and both of its skills (`terminal:tui-design`, `terminal:vhs-cli-demos`). Update later with `/plugin marketplace update gfargo-skills`.

> Non-interactive: `claude plugin marketplace add gfargo/skills && claude plugin install terminal@gfargo-skills`.

### With `npx skills`

This is a standard Claude Code marketplace, so [Vercel's `skills` CLI](https://github.com/vercel-labs/skills) installs from it too — handy on Cursor / Codex / Gemini CLI, for project-local installs, or when you only want one skill:

```bash
npx skills add gfargo/skills                        # pick from a list
npx skills add gfargo/skills --all                  # every skill in the repo
npx skills add gfargo/skills --skill vhs-cli-demos  # just one
npx skills add gfargo/skills -g -a claude-code      # global, for Claude Code
```

It resolves the skills via this repo's `.claude-plugin/marketplace.json`. Note the difference: Claude Code's `/plugin` installs a plugin whole (both skills), while `npx skills --skill` can take them individually.

---

## Skills

| Skill | Plugin | For |
|---|---|---|
| **[`terminal:tui-design`](#terminaltui-design)** | `terminal` | Designing and building clean, professional TUIs and CLIs |
| **[`terminal:vhs-cli-demos`](#terminalvhs-cli-demos)** | `terminal` | Capturing deterministic screenshots and demo GIFs of them |

I built these two to pair: design a terminal app, then capture it for your README, docs, or marketing site.

### `terminal:tui-design`

Designing clean, professional, minimal terminal UIs and command-line tools across **Go, Rust, Python, and TypeScript**.

- **Universal patterns** — the seven canonical TUI layouts, visual hierarchy in monospace, color as a semantic system (and `NO_COLOR`), cross-app keybinding conventions, and the four non-negotiables: alt screen, panic-safe restore, `SIGWINCH`, `SIGTSTP`.
- **Per-ecosystem deep-dives**, loaded only when relevant — Bubble Tea / Lipgloss / Bubbles (Go), Ratatui / Crossterm (Rust), Textual / Rich (Python), Ink / `@clack/prompts` / `@inquirer/prompts` (TypeScript).
- **Exemplar case studies** — what lazygit, k9s, btop, helix, yazi, and atuin each get right.

```bash
npx skills add gfargo/skills --skill tui-design     # just this one
```

<details>
<summary><strong>Example prompts</strong></summary>

- "Build me a TUI for browsing my Postgres tables."
- "Bubble Tea or Ratatui for this?"
- "Review this lazygit-style layout."
- "How should I lay out this CLI dashboard?"

</details>

### `terminal:vhs-cli-demos`

Deterministic screenshots and demo GIFs of any CLI or terminal app with [Charm VHS](https://github.com/charmbracelet/vhs) — pixel-accurate, scriptable, reproducible in CI.

- **Tape authoring & determinism** — pin the clock, lock the theme, settle for cold-start, plus the VHS shell gotchas that bite everyone once.
- **Motion-GIF storytelling** — a `.tape` is a screenplay: one story per demo, budgeted read-time, end on the UI not a shell prompt.
- **Lossless size optimization** — raw VHS GIFs run 10–20 MB; a bundled `gifsicle -O3` pass shrinks them 20–30× with zero quality loss, no lossy defaults.
- **Scale** — the recipe-catalog + driver pattern for keeping many captures in sync as the UI changes.

```bash
npx skills add gfargo/skills --skill vhs-cli-demos  # just this one
```

<details>
<summary><strong>Example prompts</strong></summary>

- "Make a demo GIF of my CLI for the README."
- "Screenshot my terminal app for the docs site."
- "This GIF is 30 MB and GitHub won't embed it."
- "Set up repeatable screenshots for all of my TUI's screens."

</details>

> Needs `vhs` and `gifsicle` — see [Requirements](#requirements).

---

## Requirements

- Claude Code with `/plugin` support, or any agent [`npx skills`](https://github.com/vercel-labs/skills) targets.
- For `terminal:vhs-cli-demos` captures:

  ```bash
  brew install vhs       # pulls in ffmpeg + ttyd
  brew install gifsicle  # the lossless GIF optimization step
  ```

`terminal:tui-design` has no external dependencies.

---

## Structure

A marketplace catalogs plugins; a plugin bundles skills; a skill is invoked as `<plugin>:<skill>`. Annotated against the files:

```
marketplace  gfargo/skills        .claude-plugin/marketplace.json
└── plugin   terminal             plugins/terminal/.claude-plugin/plugin.json
    ├── tui-design                plugins/terminal/skills/tui-design/      (SKILL.md + references/)
    └── vhs-cli-demos             plugins/terminal/skills/vhs-cli-demos/   (SKILL.md + references/ + scripts/)
```

---

## Managing the marketplace

```bash
/plugin marketplace list                   # browse available plugins
/plugin marketplace update gfargo-skills   # pull the latest catalog
/plugin install terminal@gfargo-skills     # install / reinstall a plugin
/plugin list                               # what you have installed
/plugin marketplace remove gfargo-skills   # remove the marketplace
```

---

## Adding a skill

When I add one:

1. Drop a `skills/<name>/` folder (`SKILL.md` plus any `references/` and `scripts/`) into the relevant plugin, or add a new plugin under `plugins/` and list it in `marketplace.json`.
2. Bump the `version` in both `marketplace.json` and that plugin's `plugin.json`.
3. Tag a release.

Issues and PRs welcome — library updates, new exemplars, and bug reports especially.

---

## References

- [Agent Skills](https://agentskills.io) — the open standard these skills follow.
- Claude Code: [skills](https://code.claude.com/docs/en/skills) · [plugins](https://code.claude.com/docs/en/plugins) · [plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) · [plugins reference](https://code.claude.com/docs/en/plugins-reference) (the `plugin.json` / `.claude-plugin/` schema).
- [`vercel-labs/skills`](https://github.com/vercel-labs/skills) — the cross-agent installer.
- Background: I wrote up [the story behind `tui-design`](https://wp.griffen.codes/2026/04/30/tui-design-skill-claude/); more on [griffen.codes](https://griffen.codes).

---

## License

[MIT](LICENSE) © Griffen Fargo
