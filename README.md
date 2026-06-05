# gfargo/skills

> A curated **[Claude Code](https://docs.claude.com/en/docs/claude-code) skills marketplace** by [Griffen Fargo](https://github.com/gfargo). Add it once, then install any skill you want — more land here over time.

[![Release](https://img.shields.io/github/v/release/gfargo/skills?label=release&color=2da44e)](https://github.com/gfargo/skills/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Skills are reusable expertise you hand to Claude. Install one and it loads **automatically**, on the right task, in every project — no copy-pasting prompts into each repo. This marketplace bundles mine into plugins you can install à la carte.

---

## Contents

- [Quick start](#quick-start)
- [Skills](#skills)
  - [`terminal:tui-design`](#terminaltui-design)
  - [`terminal:vhs-cli-demos`](#terminalvhs-cli-demos)
- [Requirements](#requirements)
- [How it works](#how-it-works)
- [Managing the marketplace](#managing-the-marketplace)
- [Repository layout](#repository-layout)
- [Adding a skill](#adding-a-skill)
- [Related reading](#related-reading)
- [License](#license)

---

## Quick start

```bash
# 1. Add the marketplace (one time)
/plugin marketplace add gfargo/skills

# 2. Install a plugin — its skills become available, namespaced by plugin
/plugin install terminal@gfargo-skills
```

That's it. The skills now load themselves whenever they're relevant; you don't invoke them by hand.

> Prefer the shell? `claude plugin marketplace add gfargo/skills` then `claude plugin install terminal@gfargo-skills` does the same thing non-interactively.

---

## Skills

| Skill | Plugin | What it's for |
|---|---|---|
| **[`terminal:tui-design`](#terminaltui-design)** | `terminal` | Design and build clean, professional TUIs and CLIs |
| **[`terminal:vhs-cli-demos`](#terminalvhs-cli-demos)** | `terminal` | Capture deterministic screenshots and demo GIFs of them |

The two pair naturally: **design** a terminal app, then **capture** it for your README, docs, or marketing site.

### `terminal:tui-design`

Design and build clean, professional, minimal terminal UIs and command-line tools across **Go, Rust, Python, and TypeScript**.

- **Universal patterns** — the seven canonical TUI layouts, visual hierarchy in monospace, color as a semantic system (and the `NO_COLOR` convention), cross-app keybinding conventions, and the four non-negotiables: alt screen, panic-safe terminal restore, `SIGWINCH`, `SIGTSTP`.
- **Per-ecosystem deep-dives** — Bubble Tea / Lipgloss / Bubbles (Go), Ratatui / Crossterm (Rust), Textual / Rich (Python), Ink / `@clack/prompts` / `@inquirer/prompts` (TypeScript). Loaded only when relevant, so the skill stays cheap until you need it deep.
- **Exemplar case studies** — what lazygit, k9s, btop, helix, yazi, atuin, and friends each get right.

<details>
<summary><strong>When it triggers · example prompts</strong></summary>

Fires automatically on terminal-UI work, even when you never say "TUI":

- "Build me a TUI for browsing my Postgres tables."
- "Should I use Bubble Tea or Ratatui for this?"
- "Review this lazygit-style layout."
- "How should I lay out this CLI dashboard?"

</details>

### `terminal:vhs-cli-demos`

Generate **deterministic screenshots and demo GIFs** of any CLI or terminal app with [Charm VHS](https://github.com/charmbracelet/vhs) — pixel-accurate, scriptable, and reproducible in CI.

- **Tape authoring & determinism** — pin the clock, lock the theme, settle for cold-start, plus the VHS shell gotchas that bite everyone exactly once.
- **Motion-GIF storytelling** — a `.tape` is a screenplay: one story per demo, budgeted read-time, end on the UI not a shell prompt.
- **Lossless size optimization** — raw VHS GIFs land at 10–20 MB; a bundled `gifsicle -O3` step shrinks them 20–30× with zero quality loss (and no lossy defaults).
- **Scale** — the recipe-catalog + driver pattern for keeping many captures in sync as your UI changes.

<details>
<summary><strong>When it triggers · example prompts</strong></summary>

Fires on capture/recording work, even without naming VHS:

- "Make a demo GIF of my CLI for the README."
- "Screenshot my terminal app for the docs site."
- "This GIF is 30 MB and GitHub won't embed it — help."
- "Set up repeatable screenshots for all of my TUI's screens."

</details>

> Needs `vhs` and `gifsicle` installed — see [Requirements](#requirements).

---

## Requirements

- **Claude Code** with plugin support (the `/plugin` commands).
- For **`terminal:vhs-cli-demos`** captures:

  ```bash
  brew install vhs       # pulls in ffmpeg + ttyd
  brew install gifsicle  # the lossless GIF optimization step
  ```

`terminal:tui-design` has no external dependencies.

---

## How it works

Claude Code skills are distributed through **plugins**, and plugins are listed by a **marketplace** (this repo):

```
marketplace  (gfargo/skills)
└── plugin   (terminal)
    └── skills  (tui-design, vhs-cli-demos)   →  used as  terminal:<skill>
```

Add the marketplace once, then install whichever plugins you want. Installing a plugin makes all of its skills available, namespaced as `<plugin>:<skill>`. Skills are **description-triggered** — Claude loads them when the task matches, so there's nothing to remember and nothing to invoke manually.

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

## Repository layout

```
.
├── .claude-plugin/
│   └── marketplace.json            # the catalog — lists every plugin
└── plugins/
    └── terminal/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            ├── tui-design/         # SKILL.md + references/
            └── vhs-cli-demos/      # SKILL.md + references/ + scripts/
```

---

## Adding a skill

1. Drop a `skills/<name>/` folder (a `SKILL.md`, plus any `references/` and `scripts/`) into the relevant plugin — or add a whole new plugin under `plugins/` and list it in `marketplace.json`.
2. Bump the `version` in both `marketplace.json` and that plugin's `plugin.json`.
3. Tag a release.

Issues and PRs are welcome — library updates, new exemplars, and bug reports especially.

---

## Related reading

- [**Teaching Claude to design TUIs**](https://wp.griffen.codes/2026/04/30/tui-design-skill-claude/) — the story behind `tui-design`.
- More write-ups at [griffen.codes](https://griffen.codes).

---

## License

[MIT](LICENSE) © Griffen Fargo
