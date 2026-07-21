# gfargo/skills

> Personal agentic coding skills — terminal tooling, VPS infrastructure management, and terminal game dev.

[![skills.sh](https://www.skills.sh/b/gfargo/skills)](https://www.skills.sh/gfargo/skills)
[![Release](https://img.shields.io/github/v/release/gfargo/skills?label=release&color=2da44e)](https://github.com/gfargo/skills/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Installable agentic coding skills following the [Agent Skills](https://agentskills.io) spec. Three plugin bundles: **terminal** (TUI design + VHS demos), **devops** (VPS stack management with strut), and **games** (terminal card games with ink-playing-cards).

## Install

### Agent Skills CLI

[`npx skills`](https://github.com/vercel-labs/skills) installs these skills globally or into a project:

```bash
npx skills add gfargo/skills                        # choose from a list
npx skills add gfargo/skills --all                  # install every skill
npx skills add gfargo/skills --skill tui-design     # terminal: TUI design
npx skills add gfargo/skills --skill vhs-cli-demos  # terminal: VHS demos
npx skills add gfargo/skills --skill strut          # devops: strut VPS management
npx skills add gfargo/skills --skill ink-playing-cards  # games: card games in the terminal
```

### Claude Code Plugin

```bash
/plugin marketplace add gfargo/skills
/plugin install terminal@gfargo-skills   # TUI design + VHS demos
/plugin install devops@gfargo-skills     # strut VPS management
/plugin install games@gfargo-skills      # ink-playing-cards card games
```

### Kiro IDE

The strut skill installs natively via the strut CLI:

```bash
strut skills install          # installs to .kiro/skills/strut/
```

Or import from GitHub: paste `https://github.com/gfargo/skills/tree/main/plugins/devops/skills/strut` in Kiro's skill import dialog.

## Plugins

| Plugin | Skills | Use It For |
|--------|--------|-----------|
| **terminal** | `tui-design`, `vhs-cli-demos` | Designing polished TUIs/CLIs, capturing screenshots and demo GIFs |
| **devops** | `strut` | Managing Docker Compose stacks on VPS — deploy, backup, drift, secrets, fleet |
| **games** | `ink-playing-cards` | Building terminal card games with Ink + React — decks, zones, events, effects |

---

### `terminal:tui-design`

Design and build professional terminal UIs across Go, Rust, Python, and TypeScript.

Covers TUI layout patterns, visual hierarchy, density, color, keybindings, focus, and terminal hygiene. Ecosystem guidance for Bubble Tea, Ratatui, Textual, Ink, Clack, Inquirer, and more.

```bash
npx skills add gfargo/skills --skill tui-design
```

---

### `terminal:vhs-cli-demos`

Create deterministic screenshots and demo GIFs of CLIs and TUIs with [Charm VHS](https://github.com/charmbracelet/vhs).

Covers tape authoring, fixed-theme captures, lossless GIF optimization with `gifsicle`, and repeatable capture pipelines.

```bash
npx skills add gfargo/skills --skill vhs-cli-demos
```

---

### `devops:strut`

Operate Docker Compose stacks on VPS infrastructure with [strut](https://github.com/gfargo/strut).

Covers deploying and releasing services, database backup/restore/rehearsal, config and image-digest drift detection, secret rotation, fleet status monitoring, domain/SSL configuration, stack validation, and VPS audit/migration.

Uses progressive disclosure: a lean router `SKILL.md` with a command quick-reference, and detailed procedures in `references/` loaded on demand.

```bash
npx skills add gfargo/skills --skill strut
# or via the strut CLI:
strut skills install
```

---

### `games:ink-playing-cards`

Build terminal card games with [ink-playing-cards](https://github.com/gfargo/ink-playing-cards), a React component library for Ink 6 (React 19 for CLIs).

Covers card components, deck management, zone systems (hands, discard pile, play area), event systems, and effect systems for building card game logic.

```bash
npx skills add gfargo/skills --skill ink-playing-cards
```

---

## Requirements

- `terminal:tui-design` — no external dependencies
- `terminal:vhs-cli-demos` — requires `vhs` and `gifsicle` (`brew install vhs gifsicle`)
- `devops:strut` — requires [strut](https://github.com/gfargo/strut) installed on the machine
- `games:ink-playing-cards` — requires `ink` and `react` (`npm install ink-playing-cards`, Node >= 20)

## Auto-Sync

Source repos are synced automatically via GitHub Actions workflows:

| Skill | Source | Workflow |
|-------|--------|----------|
| `tui-design` | [gfargo/tui-design-skill](https://github.com/gfargo/tui-design-skill) | `sync-tui-design.yml` |
| `strut` | [gfargo/strut](https://github.com/gfargo/strut) `.kiro/skills/strut/` | `sync-strut.yml` |
| `ink-playing-cards` | [gfargo/ink-playing-cards](https://github.com/gfargo/ink-playing-cards) `skills/ink-playing-cards/` | `sync-ink-playing-cards.yml` |

When a new release is published in a source repo, the corresponding sync workflow mirrors the skill content here, bumps the plugin version, and cuts a new `gfargo/skills` release.

## License

[MIT](LICENSE) © Griffen Fargo
