# The recipe-catalog pattern

One-off `.tape` files are fine for a handful of captures. But when a project needs
*many* — every view, every theme, kept in sync as the UI evolves — hand-written
tapes rot: they drift from the UI, people regenerate them by hand, and the
boilerplate gets copy-pasted. The fix is to treat captures as **data**: a typed
catalog of named scenes plus a driver that turns each into a tape, runs VHS, and
files the output. This is the pattern behind a mature capture pipeline.

This reference is language-agnostic in spirit; the concrete sketch is TypeScript/
Node because that's the common case, but the same decomposition works in any
language.

## When to adopt it

Reach for the catalog when you hit any of these:

- More than ~5–10 captures, or a matrix (views × themes).
- Captures drift out of sync with the UI because regenerating is manual.
- You're copy-pasting tape boilerplate (the same `Set` block, the same setup).
- You want `one command` to regenerate everything for a release or a docs sync.

Below that threshold, a couple of checked-in tapes are simpler — don't over-build.

## The four pieces

```
bin/screenshot/
  recipes.ts      # the catalog: a typed list of named scenes (DATA)
  tape.ts         # recipe → VHS tape string (pure transform)
  themes.ts       # map app theme/preset → VHS terminal palette
screenshot.ts     # the driver: fixture → tape → vhs → output → optimize → cleanup
syncScreenshots.ts# copy web-ready assets into the docs/site folder
```

### 1. The recipe catalog (data)

A recipe is a plain object describing one scene. Keep it declarative — no VHS
syntax leaks in here.

```ts
type Action =
  | { kind: 'type'; text: string }
  | { kind: 'key'; key: string; count?: number }
  | { kind: 'sleep'; ms: number }

type Recipe = {
  name: string                 // 'ui-history' (still) or 'demo-views' (motion)
  description: string          // surfaces in --list
  scenario: string | null      // named fixture to spin up (see below), or null
  command: string              // what to run: 'ui --view history'
  actions?: Action[]           // keystrokes after launch
  fontSize?: number            // size the capture via font, not pixels (default ~20)
  canvas?: { width: number; height: number }   // RARE: explicit px override, eyeballed
  theme?: string               // lock a preset
  emitGif?: boolean            // true → motion GIF; false/absent → still PNG
}

export const RECIPES: Recipe[] = [ /* ... */ ]
```

**Naming convention that pays off:** prefix motion recipes `demo-*` and stills
`ui-*` (or similar). Reviewers, the sync list, and `--list` can tell them apart
instantly, and you can branch behavior on it (e.g. only GIFs get the optimize
pass and the trailing-quit strip).

### 2. The tape builder (pure transform)

`recipe → tape string`. This is where all the determinism and gotcha-handling
lives **once**, so no recipe has to remember it:

- Emit the `Set` block (shell, a fixed `FontSize`, theme, `CursorBlink false`).
  Leave `Width`/`Height` to VHS's defaults so glyphs stay crisp — don't compute
  pixel dimensions from a cols×multiplier formula (it stretches the font; see the
  kerning trap in `tape-reference.md`). If a recipe needs an unusually wide or
  tall canvas, carry an optional explicit `width`/`height` on the recipe and pass
  it through verbatim, eyeballed — not derived.
- `Hide` … `Show` around setup: unquoted-`$PATH` export, pinned-time env var,
  `cd` into the fixture, `clear`.
- For stills: append `Screenshot name.png`; for GIFs: `Output name.gif`.
- **Strip a trailing quit for GIFs** so the recording ends on the UI, not a shell
  prompt; keep it for stills if your driver quits the app after the shot.
- Translate `actions[]` into `Type`/`Key`/`Sleep` lines.

### 3. Deterministic fixtures (scenarios)

Captures need realistic, *stable* state. Rather than depend on the real repo/DB,
spin up a named fixture per recipe — a throwaway git repo in a temp dir, a seeded
database, a sample project tree. The recipe references it by name (`scenario`),
the driver materializes it before recording and tears it down after. This is what
makes "a feature-branch-ready repo with 4 commits" reproducible on any machine
and in CI. (coco uses a `@gfargo/git-scenarios` package for this; the general
idea is: fixtures as named, disposable, deterministic builders.)

### 4. Theme-palette mapping

If the app has themes, the **terminal's** palette must match the app's, or every
theme renders on the same background and looks identical. Map each app preset to a
VHS palette (background/foreground + ANSI slots derived from the app's accents)
and set it in the tape. Otherwise a "theme showcase" is 30 near-identical images.

### 5. The driver

Glue, per recipe: materialize fixture → build tape → `vhs <tape>` → move output to
a stable path → **optimize GIFs losslessly** (`gifsicle -O3`) → clean up temp
files. Give it `--recipe <name>` (one), `--list`, and a no-arg "all" mode.
Critically, **put the GIF optimization in the driver**, not a manual afterthought
— otherwise the "regenerate everything" path silently re-bloats every GIF (this
is a real trap; see the size section in the main skill).

### 6. The sync step

A separate command copies only the web-ready, marketing-relevant captures into the
docs/site asset folder. Keep an explicit allow-list of which recipes are
"published" and a name-mapping (recipe name → site filename) so the catalog can
contain internal/debug captures that never ship. Run it on release or when visuals
change.

## A reference implementation

The coco CLI (`bin/screenshot/` in that repo) is a worked example of every piece
above: `recipes.ts` (catalog), `tape.ts` (builder), `terminalThemes.ts` (palette
map), `screenshot.ts` (driver, with the lossless `gifsicle -O3` step built in),
`syncScreenshots.ts` (allow-list + filename map), and a `bin/screenshot/README.md`
documenting the determinism controls and VHS gotchas. If you're building this
pattern, that README and those files are a complete, battle-tested template to
adapt.

## Determinism checklist (applies to every recipe)

- Pinned "now" so relative dates don't drift.
- Fixed `FontSize`; canvas left to VHS defaults (no computed pixel Width/Height).
- Locked theme + matching terminal palette.
- Animations/spinners off or settled before capture.
- Generous settle for interpreter cold-start before the first frame.
- GIFs: optimized losslessly in the driver; no trailing quit on camera.
