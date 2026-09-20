# PixelLab

Read this reference when a manifest selects `pixellab`, when choosing between
providers, or before any PixelLab account operation.

## Operational boundary

- Credential: `PIXELLAB_API_KEY` in `.env.local` beside the manifest or in the
  process environment. Never print or commit it.
- Status: production adapter. Generation and account workflows have live
  coverage.
- Cost unit: subscription generations. Copy the exact `pixelkiln plan` total
  into `--budget`; do not translate it into dollars.
- Account operations: balance, adopt, salvage, tag, and separately confirmed
  purge are supported. Read `docs/RECOVERY.md` before using them.

## Generator choice

| Generator | Use it for | Measured cost |
|---|---|---:|
| `map` | One prop, icon, building, or landmark, up to 400×400 | 1 generation |
| `pixflux` | Closed palettes or full-bleed backgrounds up to 400×400 | 1 generation |
| `1dir` | Reference-guided work or several candidates | 20–40 generations |
| `tiles` | Ground variations or connected structures | 20–40 generations |
| `terrain` | A two-terrain Wang tileset for elevation (grass-to-water, floor-to-cliff) | Unmeasured; borrows the same 20–40 canvas tiers |
| `imagePro` | A larger or non-square background/scene, or real style transfer | **40 generations flat**, any size |
| `character` | A character in 4 or 8 directions, its poses (`state`), and its loops (`animation`) | 1 per standard base, 6 per pro-flash base at 64px (1 from a `reference`), 20–40 per pose, 1 per template loop per direction |

`tiles` is not limited to top-down ground: `tileType` selects the projection
(`isometric` — the API default —, `oblique`, `hex`, `hex_pointy`, `octagon`,
or `square_topdown`), `tileSize` sets one tile's edge in pixels (16–128 — the
API's real ceiling; connectable sets via `tileFeature` have tighter per-shape
ranges, and square top-down roads are exactly 32), and `tileView` sets the
camera (`top-down`, `high top-down`, `low top-down`, or `side`). `outlineMode`
chooses `outline` (a dark border per tile — right for tiles meant to read as
discrete objects) or `segmentation` (no border, so a ground set tiles
seamlessly instead of quilting at every cell edge — this alone decided
whether a measured terrain set was usable). `tileFeature` asks for a
connectable set instead of independent variations: `roads` for an
18-configuration path set, `tileset` for a 16-tile Wang corner set — describe
the transition itself in the prompt ("fairway grass to rough meadow"), not
one terrain — or `building` for a floor/wall/doorway construction kit.
Passing `styleImages` overrides `tileType`/`tileView` entirely and copies
tile shape and size from the reference instead, the only way to land new
tiles on an existing sheet's ground plane.

A second layer of `tiles` parameters controls shape and depth directly,
beyond the `tileView` presets: `tileHeight` sets a non-square tile's pixel
height explicitly (16–256, e.g. 128 for a 64×128 tile) when the geometry
computed from `tileType`/`tileView` isn't what's wanted; `tileViewAngle`
(0–90 degrees) is a continuous camera angle that overrides `tileView`
entirely (0 = side, 90 = top-down); `tileDepthRatio` (0–1) overrides the
depth/thickness the view would otherwise imply; `tileFlatTopPx` (2 or 4,
`isometric` only) picks the classic 2px diamond cap versus a modern 4px one;
and `obliqueLean` (0–1, `oblique` type and building walls only) sets the
shear per pixel of height — 0.5 is a classic ~27° cabinet lean, 1.0 the full
45° diagonal `oblique` defaults to.

`tileFeature: "building"` takes its own sub-parameters: `buildingWallTiles`
(1–3, default 2) sets wall height in tiles; `buildingLayout` picks `grid`
(paints each shaped piece individually — richer, and the default for
`isometric`) or `materials` (paints flat swatches and renders pieces from
them — more consistent, and the default for `square_topdown`/`oblique`);
`buildingWallDescription`, `buildingFloorDescription`, and
`buildingFloor2Description` (each up to 500 characters) name the wall
material, floor material, and upper-storey/roof surface explicitly rather
than relying on the main prompt being split correctly (`buildingFloor2Description`
defaults to the wall material when omitted); and `buildingWallAngle`
(5–90 degrees, `square_topdown` only) sets the wall storey's own camera
angle independent of the ground pitch.

`terrain` wraps PixelLab's separate `/create-tileset` endpoint, for exactly
the case `tiles`'s own `tileFeature: "tileset"` only approximates: two named
terrain levels — `lower` (the base, e.g. water, grass, lava) and `upper`
(the elevated one, e.g. sand, dirt, stone) — connected by a Wang corner set
that tiles seamlessly. This is the shape for elevation tiers in a map: a
coastline, a dungeon floor breaking into cracked stone, a cliff edge. Unlike
every other generator, its content is not one `prompt` string: `/create-tileset`
takes `lower_description` and `upper_description` as separate fields (plus
an optional `transition_description`), so a `terrain` asset's `prompt` uses
the same numbered convention `tiles` already asks for — `"1). deep ocean
water 2). golden sandy beach 3). wet sand with foam"` — split client-side
into the three fields before the call, with the style's `promptPrefix`/
`promptSuffix` wrapped around each one individually rather than the whole
string once.

A set is always 16 tiles, or 25 when `terrainTransitionSize` is exactly 1
(the "cliff" layout, where the visual step between levels is tallest and
corners take a third "transition" value); it always resolves straight to a
finished asset with one output file per tile, never a candidate sheet to
pick from, the same as a connectable `tiles` set. `terrainTileSize` sets the
tile edge (16 or 32 in both modes, 64 needs `terrainMode: "pro"`, default
16). `terrainMode: "standard"` (the default) is the classic Wang pipeline,
optionally with `terrainShapeStyle` (`square` or `round`) for a fixed
procedural boundary; `terrainMode: "pro"` swaps that for its own shape
controls — `terrainSpreadX` (0 = steep, 1 = gradual boundary), `terrainSlopeSize`
(slope on three sides as a fraction of wall height), and `terrainRaggedness`
(0 = smooth, 1 = rough boundary noise) — and rejects `terrainShapeStyle`
outright. `terrainTransitionSize` (0, 0.25, 0.5, or 1 without `terrainShapeStyle`;
any value 0–1 with it, though above 0.5 switches to an extended 32-tile
layout pixelkiln does not yet model) controls how pronounced the elevation
step looks. `terrainView` picks `low top-down` or `high top-down` (API
default). The style's existing `outline`/`shading`/`detail` apply here too.
Reference images, a forced palette, and the `pro` pipeline's own tunables
beyond the three above (`tileStrength`, `tilesetAdherence`, and the rest)
are not modeled yet; open an issue if a real project needs one of them.

`imagePro` wraps PixelLab's Pro image tier, `/generate-image-v2` — the
`pixflux`-adjacent gap named in `pixellab-roadmap.md`: real style transfer
and non-square or larger canvases (16–792 wide, 16–688 tall; the exact
ceiling in a corner also depends on aspect ratio, e.g. 512×512 for square or
688×384 for 16:9), where `pixflux` tops out at 400×400 with no style
reference at all. Set the asset's `width`/`height` for anything other than
the style's default square. Unlike every other multi-candidate generator
here, its cost is a **flat 40 generations regardless of size** (measured;
`docs/ENDPOINTS.md`, "Single-image generators, measured") — `1dir` and
`tiles` scale with canvas area, this does not. One call still returns
several candidates to pick from by the same size tiering as `1dir` (up to
42px: 64, 43–85px: 16, 86–170px: 4, above 170px: 1), reached through the
generic background-job endpoint the same way a `revision` is. Reference
images and a style image (`reference_images`, `style_image` +
`style_options`, up to 4 subject references plus one style reference) exist
on this endpoint but are not modeled yet, and the completed job's exact
response shape has not been exercised against a live account — see
`pollImagePro` in `src/providers/pixellab.ts` if a real call ever
contradicts what it assumes.

Do not confuse pixelkiln's `map` generator with PixelLab's own "Map
Workshop": `map` returns one static prop, icon, or building in a single
generation with no scene, canvas, or placement concept. Map Workshop (scene
composition — laying out a tile floor, placing characters and movable
objects, inpainting sections in place, exporting the result) is a distinct
PixelLab product surface this adapter does not model at all.

A `map` or `pixflux` prompt for anything smaller than a standard prop
(furniture, a hand-held item, a small decoration) tends to come back
oversized relative to the scale implied by the rest of a scene by default.
PixelLab's own tutorials hit this repeatedly and fixed it the same way every
time: add an explicit size qualifier to the prompt ("a small wooden chair",
not "a wooden chair"), rather than adjusting `size` after an oversized
result comes back.

A base can start from the author's own south-facing sprite (`reference` on
the asset; standard wants it at the style's size, v3 up to 256px, pro up
to 168px), and a standard humanoid base takes `proportions` (`chibi`,
`heroic`, or multipliers) on the style or the asset. A pro base can be
designed from a `concept` image (up to 1024px) and follow the look of
another generated character in the style (`styleCharacter`), which makes
that character a dependency the way a loop's parent is.

A `reference`, `concept`, or `styleCharacter` image's own canvas size sets
the output's apparent scale, not just its content: a reference cropped or
padded to a canvas larger than the target size reliably produces an
oversized character next to everything else at the intended size. Match the
reference's canvas to the base's `size` (or the other way around) before
submitting, not after a mismatched result comes back.

A v3 loop can start from a pose image (`startFrame`) or interpolate to one
(`endFrame`), take a `subject` when the character's own description would
mislead, and ask PixelLab to `enhancePrompt`; a template loop takes
`outline`, `shading`, and `detail` overrides instead. `keepFirstFrame`
defaults to `true` on a v3 loop: the parent's resting pose (or `startFrame`,
when set) becomes frame 0 for free, so a requested `frames: 6` delivers 7
files, not 6. Account for the `+1` when checking a loop's output count or
estimating its budget; set it `false` only when the caller will supply its
own first frame downstream.

A named `template` loop (`walk`, `breathing-idle`, and similar) is trained
mostly on characters with empty hands, and reliably struggles once a
character holds an item — a weapon, a tool, anything gripped. For a
held-item character, skip the template and write a custom v3 loop with an
explicit prompt naming the held item and the motion (e.g. "knight holding a
sword, walking loop") instead of expecting the template to carry it.

A loop costs per direction, and a sprite facing one way is the sprite facing
the opposite way flipped — this holds for the east/west pair and for both
diagonal pairs, south-east/south-west and north-east/north-west. Declare the
west loop and make the east one `{ "mirror": "bot.walk.west" }`: PixelKiln
flips it locally for nothing, in the wave after the source lands. South and
north sit on the mirror line itself, so flipping either just relabels the
same asset, not a new one — generate those two directly. Generate south,
south-east, east, north-east, and north, then mirror the other three: eight
directions of one loop are then 5 generations, not 8.

A game whose camera is a fixed isometric angle typically never shows a
character facing due south/east/north/west, only the four diagonals — in
that case, declare `south-east`, `north-east`, `north-west`, and
`south-west` only and skip the cardinal four entirely, rather than
generating all 8 and mirroring the rest. `isometric` on the base (drawing
every direction and loop in isometric perspective) is an independent choice
from this and does not by itself require or imply fewer directions; it is
the game's own camera that decides how many are worth generating.

A loop that needs headroom for an effect — recoil, a muzzle flash, a
projectile, anything that extends past the subject's silhouette — needs
empty canvas space near the subject before generating; a composition with no
margin is a common cause of a failed or clipped loop, and is worth ruling
out before rewriting the prompt. Bigger pose changes want more frames: a
simple loop (a walk, a static idle) reads fine at 6; a transition that
rotates or drops the subject (a fall, a knockdown) wants 8 or more. A
multi-stage action (windup, then attack, then recovery) is a chain of
loops, not one long one: take the final frame of one loop as the next
loop's `startFrame` and link them as separate manifest assets.

A loop's `startFrame` (or the `state` it is seeded from) should already
resemble the target action, not just be a valid pose of the character.
PixelLab's own tutorials show this going wrong concretely: prompting "walk
loop" from a mid-run pose forces the model to close the loop by running,
then walking, then running again, a much harder transition than starting
from a pose that already looks like the first step of a walk.

Write the loop `prompt` (and any `subject` override) as a full descriptive
sentence of the motion — pace, energy, what leads and what follows — rather
than a terse label like "attack": PixelLab's own tutorials treat this as the
single biggest lever on animation quality, on par with frame count. If an
agent drafts the prompt from a reference image, strip any technical framing
it invents (frame rates, frame-count ranges, export terms) before
submitting — PixelLab is never told the target frame rate, only the frame
count, and a stray "24fps" or "60 to 90 frames" fragment in the prompt does
not help.

In the committed environment benchmark, PixelLab followed complex building
prompts more closely and produced broader, more separable depth bands in scenic
backgrounds. Its 256px and 384px map objects were opaque despite the route's
transparency claim, so inspect alpha on one representative result before a
batch. Inspect scenic outputs for stray marks too: one untouched 384px attempt
contained a generated signature-like glyph.

An asset that declares `revision` against a PixelLab style calls `inpaint`
(masked) or `image-to-image` (whole-image, no mask); `outpaint` is refused,
since PixelLab has no canvas-expansion endpoint. Read `docs/REVISIONS.md`'s
PixelLab section before using either — its cost is an estimate, not yet
measured. For a style aimed at a specific look (a retro/console feel or a
high-fidelity showcase asset) rather than a default, read
[pixellab-fidelity.md](./pixellab-fidelity.md) before choosing `size`,
`detail`, `shading`, or `outline`. For what PixelLab can do that this adapter
does not yet reach at all, read [pixellab-roadmap.md](./pixellab-roadmap.md)
before proposing new work.

For setup and current field constraints, use
<https://pixelkiln.griffen.codes/docs/pixellab>. When working in the PixelKiln
repository, `docs/PIXELLAB.md` and `docs/ENDPOINTS.md` are the canonical local
sources. The provider sources are <https://www.pixellab.ai/> and
<https://api.pixellab.ai/v2/docs>.
