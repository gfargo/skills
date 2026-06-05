---
name: vhs-cli-demos
description: >-
  Generate deterministic screenshots and demo GIFs of command-line and terminal
  (TUI) applications using Charm VHS. Use this skill whenever the user wants to
  capture, record, screenshot, or make a GIF/video of a CLI or terminal app —
  for a README, docs site, marketing page, changelog, release notes, or visual
  regression tests. Trigger on phrases like "record a gif of my CLI", "demo gif
  for the README", "screenshot my terminal app", "capture the TUI", "make a
  terminal recording", "VHS tape", "charm vhs", "asciinema but as a gif", or any
  request to show a terminal program in motion or as a still — even when the user
  doesn't name VHS. Also use when a captured GIF is too large and needs to be
  shrunk for the web, or when setting up a repeatable capture pipeline for many
  scenes. Covers install, authoring tapes, determinism, motion-GIF storytelling,
  and the lossless size-optimization that makes GIFs web-viable.
---

# Capturing CLI & TUI apps with VHS

[VHS](https://github.com/charmbracelet/vhs) (by Charm) drives a real PTY through a
headless terminal and records exactly what a user would see — so captures are
pixel-accurate, scriptable, and reproducible in CI. It's the right tool for
README GIFs, docs/marketing stills, release demos, and terminal visual-regression
tests. This skill is the methodology; it transfers to any CLI or TUI regardless
of language (Go, Rust, Python, Node, a shell script — VHS only sees the terminal).

The single most important thing to internalize: **a `.tape` is a screenplay, not
a config file.** You're directing a short scene — what's typed, how long each
beat holds, when the camera clicks. Two failure modes dominate, and both are
covered below: captures that look *wrong* (loading spinners, drifting dates, empty
state — a determinism problem) and GIFs that are *enormous* (10-20 MB — a
file-size problem with a one-line lossless fix).

## The essentials (if you read nothing else)

Three things separate a capture that ships from one that embarrasses you. Get
these right and the rest is polish:

1. **Always optimize GIFs losslessly.** Raw VHS GIFs are 10-20 MB of redundant
   frames. Run `gifsicle -O3` (lossless, 20-30× smaller, zero quality loss) — via
   the bundled `scripts/optimize_gif.sh` — or bake it into your pipeline. Never
   ship a raw VHS GIF, and never reach for lossy compression as the default fix.
2. **Make it deterministic — and let VHS size the canvas.** Pin the clock, lock
   the theme, pick a readable `FontSize`, and settle long enough for cold-start.
   Do **not** hand-compute `Set Width`/`Set Height` to pixel-fit a column count —
   VHS's defaults render crisply, and a mismatched canvas stretches the glyphs
   (spaced-out, broken-looking kerning). (See Determinism.)
3. **Use the right output.** `Screenshot file.png` for a single still; `Output
   file.gif`/`.mp4` for motion. `Output file.png` is a trap — it records a frame
   *sequence*, not an image.

Everything below explains the *why* and the edge cases. When in doubt, those
three are the load-bearing ones.

## When to read which reference

Keep this body in context for the workflow and the hard-won lessons. Load a
reference only when you reach that step:

- **`references/tape-reference.md`** — the full VHS tape DSL (`Set`, `Type`,
  `Sleep`, `Key`, `Hide`/`Show`, `Output`, `Screenshot`, `Source`, `Require`),
  output formats (gif/mp4/webm/png), and complete annotated example tapes for
  both a still and a motion demo. Read it when authoring a non-trivial tape or
  when you need a command you don't remember.
- **`references/recipe-catalog-pattern.md`** — how to graduate from hand-written
  one-off tapes to a generated *recipe catalog* + driver when a project needs
  many captures kept in sync (the pattern behind a mature pipeline: typed recipe
  list, scenario fixtures, theme-palette mapping, a sync step to a docs/site
  folder). Read it when the user has more than a handful of captures or wants a
  repeatable `npm run screenshot`-style workflow.

## Setup

```bash
brew install vhs        # also available via Go, Nix, apt, scoop, docker
# VHS needs ffmpeg (and ttyd) for video/gif encoding — brew pulls them in.
brew install gifsicle   # for the lossless GIF optimization step (below)
```

Verify with `vhs --version`. If `vhs new demo.tape` errors about ttyd/ffmpeg,
install those explicitly (`brew install ttyd ffmpeg`).

## The core workflow

1. **Pick the output**: a still PNG (`Screenshot`) or a motion GIF (`Output
   foo.gif`). Stills are for docs/feature shots; GIFs for workflows in motion.
2. **Write the tape** — the scene. Start from `vhs new demo.tape` or the
   annotated examples in the tape reference.
3. **Make it deterministic** — pin time, lock the theme, pick a readable font
   size, give the app time to settle (see Determinism). This is what separates a
   capture you can regenerate from one you got lucky with once.
4. **Render**: `vhs demo.tape`. Inspect the output. Iterate on timing.
5. **For GIFs, optimize losslessly** — run `scripts/optimize_gif.sh out.gif`
   (or wire `gifsicle -O3` into your pipeline). This is not optional for the web.

## Determinism — the thing that makes captures reusable

A capture is only useful if it looks identical every run. Control the sources of
drift up front:

- **Wall-clock / relative dates.** Anything that renders "3 days ago" or today's
  date will drift between runs. If the app honors an env var or flag for a fixed
  "now" (many do for exactly this reason), set it in the tape via `Env` or an
  exported variable. If it doesn't, consider adding one — it's the cleanest fix.
- **Sizing — pick a `FontSize`, then leave the canvas alone.** This is the one
  that bites: VHS sizes the output from `Set Width`/`Set Height` in *pixels*, but
  the terminal renders a *character grid* whose cell size comes from the font. If
  you hand-pick a Width/Height that doesn't land on an exact whole number of cells
  for the actual font metrics, the renderer **stretches the glyphs to fill the
  canvas** — the result looks spaced-out, with broken kerning. Don't try to
  pixel-fit a cols×rows target. Instead: set a readable `FontSize` (≈18-22) and
  **omit `Width`/`Height` so VHS uses its known-good defaults (1200×600)**, which
  render crisply. Only set explicit dimensions when you need a specific canvas
  across a set of captures — and then use round numbers with a comfortable font,
  and *look at the output* to confirm the glyphs are tight, never trust a
  cols×multiplier formula. (Determinism is preserved either way: same FontSize +
  same content → same render.)
- **Lock the theme.** Pass the app's theme flag and set VHS's terminal palette to
  match (`Set Theme`). Don't rely on ambient terminal colours.
- **Disable animations during the shot.** Spinners, idle-tip rotations, blinking
  cursors (`Set CursorBlink false`) all introduce frame-to-frame noise. Wait for
  loading states to settle before the `Screenshot`.
- **Settle for interpreter cold-start.** Inside VHS the app boots in a *fresh*
  process; interpreted runtimes (tsx/ts-node, Python, a slow node entrypoint)
  can take 2-3s to cold-start versus ~500ms warm, plus any async data load. If
  shots show "loading…" or empty state, the fix is almost always **more sleep
  before the first capture**, not a different command.

## VHS shell-environment gotchas

VHS spawns a clean shell that does **not** inherit your parent environment. These
bite everyone once:

- **Unquoted `$PATH` in exports.** `Type` types literal characters. If you write
  `Type "export PATH='...:$PATH'"`, the single quotes make bash treat `$PATH` as
  a literal string and the shell loses `git`, `sleep`, and friends. Export with
  the value unquoted so `$PATH` expands: `export PATH=/your/bin:$PATH`.
- **Forward the env you need.** API keys, tokens, or config the app reads aren't
  present in the VHS shell. Export them inside the tape (use `Hide`/`Show` so the
  export lines don't appear on camera) or pass them through your driver.
- **`Screenshot` vs `Output`.** `Output "x.png"` records the *whole session* as a
  frame sequence (a directory) — not a single image. For one still frame use
  `Screenshot x.png` (bare filename). `Output "x.gif"` / `"x.mp4"` is for motion.
- **cwd binding.** VHS's `cd`/`Type "cd …"` changes the shell cwd, but some apps
  bind their working context another way (e.g. a `--repo`/`--cwd` flag that calls
  `chdir` internally). Pass the explicit flag rather than trusting cwd.
- **macOS temp-dir symlinks.** `/var/folders/...` is really `/private/var/...`;
  tools with path-safety checks (e.g. git's `safe.directory`) can trip on the
  symlinked form. Resolve the real path before handing it to the app.

## Stills (PNG)

Keep them boring and crisp: launch, **settle generously** (a still has no second
chance — a few seconds is fine, you only emit one frame), drive any keystrokes to
reach the state you want, hold briefly, then `Screenshot name.png`. One scene per
still. If the image looks wrong, increase the settle before reaching for anything
else.

## Motion GIFs — direct a short story

A GIF is a screenplay. The discipline that makes them good (and small):

- **One story per demo.** Make a single point and stop. "The list changes per
  view" → show it on two views, done. Resist tacking on extra beats; each one
  costs viewer attention *and* bytes.
- **Show contrast, not completeness.** Two cases proving a behavior beats six
  enumerating it. Full-screen takeovers (help overlays, pickers) are tempting but
  dilute the point and balloon the file (see File size).
- **Budget read time.** Sleep long enough to *read* each beat: ballpark
  ~2.5s after opening something with text to read, ~1.2s for a view/screen
  switch, ~0.5s between quick keystrokes. Too fast is unreadable; too slow drags
  and grows the file.
- **Shorter settle than stills.** GIFs record from boot, so early "loading"
  frames read as natural startup — a ~1.5s lead-in is plenty.
- **Type multi-key sequences as one action** where the app expects them together
  (e.g. a chord); the brief intermediate state on camera often *helps* by showing
  the relationship between keys.
- **End on the UI, not a shell prompt.** Don't film a trailing quit/`q` — let the
  recording end on the last meaningful frame. If your pipeline appends a quit for
  stills, strip it for GIFs.

## File size — the gotcha that ships 18 MB GIFs

This is the lesson that bites hardest, so it gets its own section. **VHS writes
full, undeduplicated frames.** A 10-second terminal demo where almost nothing
changes between frames still lands at **10-20 MB raw** — far too heavy for a web
page or a README. Three levers, in order of impact:

1. **Optimize losslessly — always.** Run `gifsicle -O3` on the output. This is
   *lossless* inter-frame transparency optimization (no `--lossy`, no colour
   quantization): it rewrites only the pixels that change between frames.
   Typically a **20-30× reduction with zero visible difference** — e.g. a real
   demo went 15 MB → 0.4 MB. Use the bundled `scripts/optimize_gif.sh` (best-
   effort: skips with a hint if gifsicle is absent), or wire `gifsicle -O3
   --batch <file>` into your capture pipeline so regenerations stay small without
   anyone remembering a manual step. **Do this in the pipeline, not by hand** —
   any "regenerate all captures" command will otherwise re-bloat everything.
2. **Trim the story.** Less duration and fewer full-screen redraws mean
   fewer/cheaper frames *before* optimization even runs. Cutting one unnecessary
   full-overlay beat took that same demo 19 MB → 13 MB on its own; gifsicle then
   finished the job.
3. **Shrink dimensions — last resort.** Fewer pixels = smaller file, but it costs
   legibility and breaks visual consistency across a set of demos. Only reach for
   it after the first two.

Rule of thumb: **author for the story and timing; let the lossless pass handle
the bytes.** If a GIF is still multi-MB *after* `-O3`, the recipe is doing too
much — tighten the scene rather than reaching for `--lossy` and degrading
quality. (If you genuinely need lossy compression for an extreme case, make it an
explicit, opt-in choice the user asked for — never the silent default.)

## Scaling up: from one-off tapes to a catalog

A handful of captures can be hand-written tapes checked into `assets/` or
`docs/`. Once a project needs *many* — every view, every theme, kept in sync as
the UI changes — graduate to a **recipe catalog + driver**: a typed list of named
scenes, a driver that spins up fixtures, generates the tape, runs VHS, and a sync
step that copies web-ready assets into the docs/site. Read
`references/recipe-catalog-pattern.md` for the full pattern and a reference
implementation. Signs it's time: people regenerate captures by hand, shots drift
out of sync with the UI, or you're copy-pasting tape boilerplate.

## Use cases

- **README / docs**: a hero GIF plus per-feature stills. Keep GIFs short and
  optimized; stills for anything users need to read.
- **Marketing site**: same captures, synced into the site's asset folder. Animate
  the headline workflows; use stills for grids and theme showcases.
- **Visual regression**: deterministic stills are diff-able (`pixelmatch`,
  `odiff`) against a committed baseline. Run as a manual/release CI job, not on
  every push — rendering is too slow for the hot path but invaluable at release.
