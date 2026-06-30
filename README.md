# gfargo/skills

> Personal agentic coding skills, starting with terminal tooling.

[![skills.sh](https://www.skills.sh/b/gfargo/skills)](https://www.skills.sh/gfargo/skills)
[![Release](https://img.shields.io/github/v/release/gfargo/skills?label=release&color=2da44e)](https://github.com/gfargo/skills/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

This repo publishes installable agentic coding skills that follow the [Agent Skills](https://agentskills.io) spec. The current collection is focused on terminal apps: designing polished TUIs and capturing clean demos of them.

## Install

### Agent Skills CLI

[`npx skills`](https://github.com/vercel-labs/skills) installs these skills globally or into a project:

```bash
npx skills add gfargo/skills                        # choose from a list
npx skills add gfargo/skills --all                  # install every skill
npx skills add gfargo/skills --skill tui-design     # install one skill
npx skills add gfargo/skills --skill vhs-cli-demos  # install one skill
```

### Claude Code Plugin

```bash
/plugin marketplace add gfargo/skills
/plugin install terminal@gfargo-skills
```

Claude Code can also install this repo as a plugin marketplace. The command above installs the `terminal` plugin, including both `terminal:tui-design` and `terminal:vhs-cli-demos`.

To update later:

```bash
/plugin marketplace update gfargo-skills
```

For non-interactive setup:

```bash
claude plugin marketplace add gfargo/skills
claude plugin install terminal@gfargo-skills
```

## Skills

| Skill | Use It For |
|---|---|
| [`terminal:tui-design`](#terminaltui-design) | Designing and building clean TUIs and CLIs |
| [`terminal:vhs-cli-demos`](#terminalvhs-cli-demos) | Capturing deterministic screenshots and demo GIFs of terminal apps |

These two are meant to pair well: design a terminal interface, then capture it for a README, docs site, release note, or marketing page.

### `terminal:tui-design`

Design and build professional terminal UIs across Go, Rust, Python, and TypeScript.

It covers:

- TUI layout patterns, visual hierarchy, density, color, keybindings, focus, and terminal hygiene.
- Ecosystem-specific guidance for Bubble Tea, Ratatui, Textual, Ink, Clack, Inquirer, and related tools.
- Practical review heuristics for making terminal apps feel less cramped, noisy, or fragile.
- Case studies from apps like lazygit, k9s, btop, helix, yazi, fzf, and atuin.

Example prompts:

- "Build me a TUI for browsing my Postgres tables."
- "Bubble Tea or Ratatui for this?"
- "Review this lazygit-style layout."
- "How should I lay out this CLI dashboard?"

Install only this skill:

```bash
npx skills add gfargo/skills --skill tui-design
```

### `terminal:vhs-cli-demos`

Create deterministic screenshots and demo GIFs of CLIs and TUIs with [Charm VHS](https://github.com/charmbracelet/vhs).

It covers:

- Tape authoring for still screenshots, GIFs, MP4s, and reusable capture scenes.
- Deterministic captures: fixed themes, stable clocks, readable sizing, cold-start settling, and clean environments.
- Lossless GIF optimization with `gifsicle`, including a bundled helper script.
- Repeatable capture pipelines for projects that need many demos kept in sync.

Example prompts:

- "Make a demo GIF of my CLI for the README."
- "Screenshot my terminal app for the docs site."
- "This GIF is 30 MB and GitHub won't embed it."
- "Set up repeatable screenshots for all of my TUI's screens."

Install only this skill:

```bash
npx skills add gfargo/skills --skill vhs-cli-demos
```

## Requirements

`terminal:tui-design` has no external dependencies.

For `terminal:vhs-cli-demos`, install VHS and `gifsicle`:

```bash
brew install vhs
brew install gifsicle
```

## About

I'm [Griffen Fargo](https://griffen.codes). This is where I publish the skills I build and use for agentic coding workflows.

Issues and PRs are welcome, especially for library updates, new terminal exemplars, and fixes to stale guidance.

## License

[MIT](LICENSE) © Griffen Fargo
