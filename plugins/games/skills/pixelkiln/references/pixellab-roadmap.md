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
- **Pro-tier full-bleed background generation** (`generate-image-v2`, PixelLab's
  Pro image tier) at arbitrary custom aspect ratios (not just square) —
  already measured at 40 generations in `docs/ENDPOINTS.md`, but `pixflux`
  wraps only the cheap 1-generation non-Pro endpoint. Demonstrated
  generating a 384×216 scene background in "How to Make Animated Pixel Art
  Scenes with PixelLab." Closed by the `imagePro` generator; see
  `pixellab.md`. Reference images and a style image
  (`reference_images`/`style_image`) remain unmodeled, and the completed
  job's response shape has not itself been exercised live yet.
- **A standalone single isometric tile** (`/create-isometric-tile`, a third,
  separate path to isometric content distinct from both `tiles`'
  `tileType: "isometric"` and `terrain`'s `/create-tileset`, which is
  confirmed square-only with no isometric option at all) — a real gap: no
  per-tile elevation primitive existed for a raised mesa or a cliff block
  outside a connected ground set. Found via direct API/schema investigation,
  not a tutorial (real downstream demand: a Godot game's terrain-verticality
  work). Closed by the `isometricTile` generator; see `pixellab.md`. Its
  `isometricTileShape` (thin/thick/block) is a direct thickness control
  `tiles`/`terrain` do not have. **Cost measured once, and it corrected a
  wrong assumption**: the endpoint's own OpenAPI response schema example is
  `{ type: "usd", usd: 0.02 }`, which reads as real-dollar billing, but a
  live call against a subscription account billed exactly 1 generation
  instead — `costUnit` is `"generations"` here, not `"usd"`. Only one
  size/shape combination (32px, `"block"`) has actually been measured; no
  size-tiering formula is documented the way `1dir`/`tiles` have one. Style
  images, `init_image`/`init_image_strength`, and `color_image` remain
  unmodeled.
- **Object Creator's rotation/state/animation family** (`/create-object-pro-flash`,
  a skeleton-free entity distinct from both `character` and the generic
  `/objects` polling resource `1dir`/`map` already use) — demonstrated in
  "Object Creator" and used throughout "GBA-Style Sprites" and "Build a Game
  with AI"; `map` (one generation, no direction/state concept) and `1dir`
  (one direction only) were the closest analogs, and neither modeled
  rotation, states, or animation for a plain object. Closed by the
  `objectPro` generator; see `pixellab.md`. It reuses `character`'s own
  `asset.state`/`asset.animation` authoring shapes and mirror handling
  rather than inventing parallel ones. **Cost is not independently
  measured**: assumed identical to `character` pro-flash's own measured
  formula, since the request bodies are near-identical minus `template_id` —
  a real assumption pending a live check, not a confirmed number the way
  `isometricTile`'s now is. Batch "pack" generation (N distinct objects from
  one call) remains unmodeled — see above.
- **Pixel correction** (`/correct-pixelart`, a dedicated cleanup pass distinct
  from `image-to-pixelart` — the latter is documented, docs/ENDPOINTS.md, as
  being "for photographs and 3-D renders, not for reprocessing" pixel art)
  and **reduce colors** (`/reduce-colors`, palette-lock/quantize with an
  explicit color count or a lifted palette image, distinct from PixelKiln's
  internal `quantize()`, which only runs inside the `refine`/quality-gate
  pipeline against a manifest-declared palette) — both demonstrated
  repeatedly across this batch's character/animation tutorials as a
  near-mandatory cleanup step. Closed by the `revision` asset shape's
  `correct-pixelart` and `reduce-colors` modes; see
  `pixellab.md` and `docs/REVISIONS.md`. **Neither endpoint's cost is
  measured against a live account**: both are schema-only here, taken from
  PixelLab's live OpenAPI document rather than an observed call, and the
  schema's own dollar-denominated `usage` example is exactly the kind of
  claim this catalog has repeatedly found wrong once measured
  (`isometricTile`, `objectPro`). **Batch/multi-frame input remains
  unmodeled**: both endpoints are built to take several frames in one call so
  an animation or a character's eight directions share one consistent
  palette/cleanup pass, which is the actual differentiator the tutorials
  demonstrate — PixelKiln only ever sends one frame per revision today, the
  same limit `image-to-image`/`inpaint` already have.
- **Generic (non-character) animation and interpolation** (`/animate-with-text-v3`,
  `/animate-pixminimax`) — PixelLab's **Animate with text** tools work on
  *any* image, not just a `character`/`objectPro` asset. Demonstrated
  animating a treasure chest opening, a campfire's flames, a tree catching
  fire, a two-character combat scene, a day/night background transition, and
  chaining a sequence by feeding one animation's final frame back in as the
  next one's reference (repeatedly, in "Animate with Text," "Generate Pixel
  Animations," "New PixelLab Tool: Animate Between 2 Frames," and "How to
  Make Animated Pixel Art Scenes with PixelLab"). Closed by the `revision`
  asset shape's `animate` and `animate-pixminimax` modes; see
  `pixellab.md`. `lastFrame` covers both the "pin an ending pose" and the
  "chain from a prior animation's final frame" cases — it is just a
  manifest-relative image, which can be any earlier asset's own generated
  output file. **Neither endpoint's cost or completed-response shape is
  measured against a live account** — cost borrows `character`'s own
  measured v3-loop formula, and the frame list is extracted defensively from
  several plausible field names rather than one confirmed shape, the way
  `pollRevision` already handles `image-to-image`/`inpaint`. The dedicated
  **Interpolate** endpoint (`/interpolation-v2`, a separate "Pro" two-keyframe
  tool distinct from `last_frame` pinning on the animate endpoints above)
  remains unwrapped — a narrower, real gap, not yet investigated.

## Object Creator: batch generation only

Object Creator's "pack" generation — one prompt or style reference → N
distinct objects in one call, each with an optional per-item text override —
remains unmodeled. Demonstrated in "Object Creator" and used throughout
"GBA-Style Sprites" and "Build a Game with AI." The rest of Object Creator
(8-direction rotation, states, pro/v3 animation) is closed; see below.

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
