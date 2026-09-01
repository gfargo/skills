# gfargo/skills

> Personal agentic coding skills — terminal tooling, VPS infrastructure management, web security, and game-development workflows.

[![skills.sh](https://www.skills.sh/b/gfargo/skills)](https://www.skills.sh/gfargo/skills)
[![Release](https://img.shields.io/github/v/release/gfargo/skills?label=release&color=2da44e)](https://github.com/gfargo/skills/releases)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

Installable agentic coding skills following the [Agent Skills](https://agentskills.io) spec. Four plugin bundles: **terminal** (TUI design + VHS demos), **devops** (VPS stack management with strut), **security** (WAF management with Doorman), and **games** (terminal card games with ink-playing-cards + pixel-art workflows with PixelKiln).

## Install

### Agent Skills CLI

[`npx skills`](https://github.com/vercel-labs/skills) installs these skills globally or into a project:

```bash
npx skills add gfargo/skills                        # choose from a list
npx skills add gfargo/skills --all                  # install every skill
npx skills add gfargo/skills --skill tui-design     # terminal: TUI design
npx skills add gfargo/skills --skill vhs-cli-demos  # terminal: VHS demos
npx skills add gfargo/skills --skill strut          # devops: strut VPS management
npx skills add gfargo/skills --skill doorman        # security: WAF management
npx skills add gfargo/skills --skill ink-playing-cards  # games: card games in the terminal
npx skills add gfargo/skills@pixelkiln              # games: reproducible pixel-art workflows
```

### Claude Code Plugin

```bash
/plugin marketplace add gfargo/skills
/plugin install terminal@gfargo-skills   # TUI design + VHS demos
/plugin install devops@gfargo-skills     # strut VPS management
/plugin install security@gfargo-skills   # Doorman WAF management
/plugin install games@gfargo-skills      # ink-playing-cards + PixelKiln
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
| **security** | `doorman` | Managing Vercel & Cloudflare WAF rules as code — firewall rules, IP blocking, bot protection |
| **games** | `ink-playing-cards`, `pixelkiln` | Building terminal card games and reproducible pixel-art asset pipelines |

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

### `security:doorman`

Manage Vercel and Cloudflare WAF firewall rules as code with [Doorman](https://github.com/gfargo/doorman).

Covers creating rules, IP blocking, rate limiting, bot protection, geo-blocking, syncing local config to providers, validating configurations, exporting documentation, CI/CD automation, and translating rules between Vercel and Cloudflare formats.

Uses progressive disclosure: a lean router `SKILL.md` with a command quick-reference, and detailed procedures in `references/` loaded on demand.

```bash
npx skills add gfargo/skills --skill doorman
```

---

### `games:ink-playing-cards`

Build terminal card games with [ink-playing-cards](https://github.com/gfargo/ink-playing-cards), a React component library for Ink 6 (React 19 for CLIs).

Covers card components, deck management, zone systems (hands, discard pile, play area), event systems, and effect systems for building card game logic.

```bash
npx skills add gfargo/skills --skill ink-playing-cards
```

---

### `games:pixelkiln`

Plan, generate, review, recover, audit, pack, and export manifest-driven pixel-art projects with [PixelKiln](https://github.com/gfargo/pixelkiln).

The skill keeps paid generation budgeted and recoverable, preserves human review, and treats manifests, lockfiles, generated assets, and provenance as build state.

```bash
npx skills add gfargo/skills@pixelkiln
```

---

## Requirements

- `terminal:tui-design` — no external dependencies
- `terminal:vhs-cli-demos` — requires `vhs` and `gifsicle` (`brew install vhs gifsicle`)
- `devops:strut` — requires [strut](https://github.com/gfargo/strut) installed on the machine
- `security:doorman` — requires `@gfargo/doorman` (`npm install -g @gfargo/doorman`, Node >= 20) and provider API tokens
- `games:ink-playing-cards` — requires `ink` and `react` (`npm install ink-playing-cards`, Node >= 20)
- `games:pixelkiln` — requires PixelKiln (`npm install pixelkiln`, Node >= 20) and provider credentials for live generation

## Auto-Sync

Source repos are synced automatically via GitHub Actions workflows:

| Skill | Source | Workflow |
|-------|--------|----------|
| `tui-design` | [gfargo/tui-design-skill](https://github.com/gfargo/tui-design-skill) | `sync-tui-design.yml` |
| `strut` | [gfargo/strut](https://github.com/gfargo/strut) `.kiro/skills/strut/` | `sync-strut.yml` |
| `doorman` | [gfargo/doorman](https://github.com/gfargo/doorman) `skills/doorman/` | `sync-doorman.yml` |
| `ink-playing-cards` | [gfargo/ink-playing-cards](https://github.com/gfargo/ink-playing-cards) `skills/ink-playing-cards/` | `sync-ink-playing-cards.yml` |
| `pixelkiln` | [gfargo/pixelkiln](https://github.com/gfargo/pixelkiln) `skills/pixelkiln/` | `sync-pixelkiln.yml` |

When a new release is published in a source repo, the corresponding sync workflow mirrors the skill content here, bumps the plugin version, and cuts a new `gfargo/skills` release.

## License

[MIT](LICENSE) © Griffen Fargo
