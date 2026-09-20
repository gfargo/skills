# PixelLab surface this adapter does not cover

A catalog of PixelLab capabilities confirmed absent from PixelKiln, built by
reading all 26 tutorials on PixelLab's official YouTube channel (the 12
featured on pixellab.ai plus 14 more from the channel's "PixelLab Tutorial"
playlist, filtered to the last ~12 months) against this adapter's actual
source. Read this when scoping new PixelLab work, so effort goes toward a
confirmed gap with real demand rather than a guess. Nothing here is a
commitment to build it — it is what is missing, with the evidence for why it
might matter, so that decision can be made deliberately.

Each entry names the PixelLab tool, why pixelkiln doesn't have it today, and
which tutorial(s) demonstrated real (not hypothetical) demand for it.

## Already closed since this catalog's research began

- **Masked inpainting and whole-image editing** — was the single largest gap
  found (routine in at least 6 of the 26 tutorials). Now the `revision` asset
  shape's `inpaint` and `image-to-image` modes; see `docs/REVISIONS.md`.
  Both are confirmed live at their floor size (20 generations each, response
  shape matched on the first try for both — though the two shapes differ,
  singular for one and array-wrapped for the other). `inpaint` is further
  confirmed at its ceiling size too (512×512, exactly 40 generations), and
  further confirmed to have a real middle (25-generation) tier — 288×288
  (82944px²) and 320×320 (102400px²) both billed 25 — just positioned far
  higher than the borrowed 1024px²/2048px² breakpoints assume: 40×40,
  128×128, and 256×256 (up to 65536px²) still billed 20, not 25, while
  352×352 (123904px²) already billed 40. Both breakpoints are now tightly
  bracketed: 20→25 in (65536px², 82944px²] (a 1.27x range), and 25→40 in
  (102400px², 123904px²] (a 1.21x range). `image-to-image` above its own
  floor remains completely unmeasured — see that doc's PixelLab section
  before relying on either endpoint's cost outside what's stated here.
- **`tiles` generator parameters** (`tileType`, `tileSize`, `tileView`,
  `tileFeature` including a building-kit construction set, `outlineMode`) —
  already implemented in code, was simply undocumented in
  [pixellab.md](./pixellab.md) until this pass. Not a gap; a docs fix.
- **Diagonal character mirroring** (southeast/southwest, northeast/northwest,
  not just east/west) — already implemented in code
  (`MIRRORED_DIRECTION` in `src/types.ts`), was undocumented. Not a gap.
- **Two-terrain elevation tilesets** (`/create-tileset`, the top-down family)
  — a real gap, not a docs fix: `tiles`'s own `tileFeature: "tileset"` draws
  one terrain's edges, not two named, connected terrain levels with their own
  transition prompt. Closed by the `terrain` generator; see
  `pixellab.md`. Cost is unmeasured (borrows the same 20/25/40 canvas tiers
  already confirmed for `tiles` and `1dir`); `/create-tileset-sidescroller`,
  reference images, `color_image`, and the `pro` pipeline's own tunables
  beyond `spreadX`/`slopeSize`/`raggedness` remain unmodeled.

## Generic (non-character) animation and interpolation

PixelLab's **Interpolate** and **Animate with text** tools work on *any*
image — a standalone object, a full scene, a portrait — not just a
`character` asset. Demonstrated animating a treasure chest opening, a
campfire's flames, a tree catching fire, a two-character combat scene, a
day/night background transition, and chaining a sequence by feeding one
animation's final frame back in as the next one's reference (repeatedly, in
"Animate with Text," "Generate Pixel Animations," "New PixelLab Tool: Animate
Between 2 Frames," and "How to Make Animated Pixel Art Scenes with
PixelLab"). Pixelkiln's only interpolation concept is the `character` v3
loop's `startFrame`/`endFrame`, scoped strictly inside a `character` asset —
there is no way to animate a `map`, `pixflux`, or `tiles` output at all.

## Object Creator

Generic 8-direction object rotation, object "states" (prompted variations of
an existing object), object animation (with a pro-vs-V3 model choice), and
batch "pack" generation (one prompt or style reference → N distinct objects,
each with an optional per-item text override) — all scoped to a standalone
object, not a `character`. Demonstrated in "Object Creator" and used
throughout "GBA-Style Sprites" and "Build a Game with AI." Pixelkiln's
closest generator is `map` (single prop, one generation, no direction or
state concept at all) and `1dir` (implies one direction). Neither models
rotation, states, or animation for a plain object.

## UI elements and RPG UI kits

A dedicated UI tool: lay out multiple elements on a canvas, generate a whole
themed sheet from one prompt, split into individual assets, apply nine-slice
to scalable frames, add prompted "states" (an empty vs. full health bar), and
a separate style-reference-driven batch mode for icon sets (paste one image,
prompt a comma-list of distinct items, get back N stylistically matched
icons in one call). Demonstrated in "Create an RPG UI Set" and "Easiest way
to create pixel art UI." No pixelkiln analogue at all. If ever built, the
"states" mechanic should reuse `character`'s existing `state` vocabulary
(prompt-driven variation of a base asset) rather than invent new terms — the
UX is functionally identical. The batch icon-from-style-reference pattern
also doesn't fit `map` (one icon, one generation) or `1dir` (candidates of
one subject, not N different subjects) — it would need its own shape.

## Fonts

`create_font` exists on PixelLab's live MCP surface; no tutorial in this
batch demonstrated it in depth, so demand is unconfirmed beyond the tool's
existence. Not modeled at all.

## Map Workshop (scene composition)

PixelLab's actual scene-building surface: lay out a tile floor, place
characters into the scene purely for in-canvas scale reference ("really
important... without it, it's very easy to accidentally generate furniture
that's way too big or way too small" — Create Interior Maps), in-paint
sections in place (with a documented two-tier cost: a "default" model at 1
credit, a "pro" model that costs more but handles complex prompts better),
place movable/duplicable objects distinct from permanent in-painting, extend
an existing tileset by anchoring one terrain slot to what's already there and
describing only the new terrain, and export the whole map (tilesets, full
composite, and every object as separate files). Demonstrated at length in
"Create Interior Maps," "PixelLab Map Workshop Tutorial," "How to Create
Destructible Environments in Seconds," and "Create a Full Side Scroller
Level." Zero pixelkiln analogue — and see pixellab.md for the naming
collision with pixelkiln's own unrelated `map` generator.

One concrete, recurring gotcha worth carrying into any future implementation:
default/generic object prompts in Map Workshop reliably come out oversized
relative to the placed scale reference; appending an explicit size qualifier
("small") is the demonstrated fix, repeated across a table, a floor lamp, a
fridge, and a bed in "Create Interior Maps" alone.

PixelLab's own official Map-Workshop-to-Godot export ("Export Your PixelLab
Map to Godot in 3 Minutes") needs a third-party community plugin and manual
tile-size entry. PixelKiln's independent Godot exporter
(`pack --format godot`, `src/pipeline/tileset-export.ts`) already writes a
native Godot 4 `TileSet` resource directly, verified in CI — worth knowing if
Map Workshop parity is ever considered, since the export leg is arguably
already solved better here than in PixelLab's own reference flow.

## Skeleton-driven animation and animation-to-animation motion transfer

Two more animation mechanisms with no equivalent: **"Animate with skeleton"**
(fit a rig template — bipedal, realistic, chibi, or quadruped body plans —
generate two frames at a time with each accepted pair becoming context for
the next, height/head-size/offset controls) and **"Animation to animation"**
(transfer the motion/shape structure of an *existing* walk cycle onto a
newly described or referenced character, frame budget capped by the source's
canvas size). Both demonstrated in "How to Make Walking Animations for Pixel
Art Characters in PixelLab." Neither exists in pixelkiln's `character`
schema, which only knows named templates, v3 text loops, and pro loops — none
take a user-supplied motion source or expose a skeleton-editing surface.
Already flagged as explicitly out of scope in docs/PIXELLAB.md
("skeleton-driven animation"); animation-to-animation is new information not
previously catalogued anywhere.

## Pixel correction

A dedicated cleanup pass distinct from `image-to-pixelart`: takes an
already-generated image and a strength slider, reduces noise/color
complexity while preserving detail at low strength. Demonstrated in "How to
Make Animated Pixel Art Scenes with PixelLab," used specifically because
`image-to-pixelart` is documented (docs/ENDPOINTS.md) as being "for
photographs and 3-D renders, not for reprocessing" pixel art — this tool is
built for exactly the reprocessing case that warning excludes.

## Reduce colors

Palette-lock/quantize on an already-generated image or animation: pick a
target color count or supply an exact palette image, "process all frames" of
an animation in one pass, with a dithering option. Used as a near-mandatory
last step in nearly every character/animation tutorial in this batch (e.g.,
"How To Create Isometric Animals," "How to Create Destructible Environments
in Seconds," "How to Make Animated Pixel Art Scenes"). This is distinct from
PixelKiln's internal `quantize()`, which is used only inside the
`refine`/quality-gate pipeline against a manifest-declared palette, not
exposed as a general ad hoc post-generation cleanup step.

## Pro-tier full-bleed background generation

`generate-image-v2`, PixelLab's Pro image tier, at arbitrary custom aspect
ratios (not just square) — already measured at 40 generations in
docs/ENDPOINTS.md, but pixelkiln's `pixflux` generator wraps only the cheap
1-generation non-Pro endpoint. Demonstrated generating a 384×216 scene
background in "How to Make Animated Pixel Art Scenes with PixelLab." No
`pixflux`-adjacent way to reach the Pro tier or a non-square canvas today.

## Not PixelLab gaps at all — different products, no action implied

- **PixelLab's Game Builder** (hosted web IDE, its own chat agent, git-backed
  projects) and its **`agent_*` MCP tools** (`agent_list`/`agent_talk`/
  `agent_inspect`/`agent_feedback`/`agent_help`) manage a *separate* PixelLab
  product — "deploy your own agent," confirmed directly from the live tool
  schemas — unrelated to game-asset generation. Nothing to build here.
- Portraits, outfit transfer, and lip-sync/vocal-animation/talking-gif are
  already named as out of scope in docs/PIXELLAB.md. Portrait and
  outfit-transfer support exist on an internal branch stack not yet merged;
  lip-sync and vocal animation have no in-flight work.
