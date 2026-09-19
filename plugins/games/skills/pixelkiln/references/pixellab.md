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
| `pixflux` | Closed palettes or full-bleed backgrounds | 1 generation |
| `1dir` | Reference-guided work or several candidates | 20–40 generations |
| `tiles` | Ground variations or connected structures | 20–40 generations |
| `character` | A character in 4 or 8 directions, its poses (`state`), and its loops (`animation`) | 1 per standard base, 6 per pro-flash base at 64px (1 from a `reference`), 20–40 per pose, 1 per template loop per direction |

`tiles` is not limited to top-down ground: `tileType` selects the projection
(`isometric` — the API default —, `oblique`, `hex`, `hex_pointy`, `octagon`,
or `square_topdown`), `tileSize` sets one tile's edge in pixels (16–256), and
`tileView` sets the camera (`top-down`, `high top-down`, `low top-down`, or
`side`). `outlineMode` chooses `outline` (a dark border per tile — right for
tiles meant to read as discrete objects) or `segmentation` (no border, so a
ground set tiles seamlessly instead of quilting at every cell edge — this
alone decided whether a measured terrain set was usable). `tileFeature` asks
for a connectable set instead of independent variations: `roads` for an
18-configuration path set, `tileset` for a 16-tile Wang corner set — describe
the transition itself in the prompt ("fairway grass to rough meadow"), not
one terrain — or `building` for a floor/wall/doorway construction kit.
Passing `styleImages` overrides `tileType`/`tileView` entirely and copies
tile shape and size from the reference instead, the only way to land new
tiles on an existing sheet's ground plane.

Do not confuse pixelkiln's `map` generator with PixelLab's own "Map
Workshop": `map` returns one static prop, icon, or building in a single
generation with no scene, canvas, or placement concept. Map Workshop (scene
composition — laying out a tile floor, placing characters and movable
objects, inpainting sections in place, exporting the result) is a distinct
PixelLab product surface this adapter does not model at all.

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

A loop costs per direction, and a sprite facing one way is the sprite facing
the opposite way flipped — this holds for the east/west pair and for both
diagonal pairs, south-east/south-west and north-east/north-west. Declare the
west loop and make the east one `{ "mirror": "bot.walk.west" }`: PixelKiln
flips it locally for nothing, in the wave after the source lands. South and
north sit on the mirror line itself, so flipping either just relabels the
same asset, not a new one — generate those two directly. Generate south,
south-east, east, north-east, and north, then mirror the other three: eight
directions of one loop are then 5 generations, not 8.

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
measured.

For setup and current field constraints, use
<https://pixelkiln.griffen.codes/docs/pixellab>. When working in the PixelKiln
repository, `docs/PIXELLAB.md` and `docs/ENDPOINTS.md` are the canonical local
sources. The provider sources are <https://www.pixellab.ai/> and
<https://api.pixellab.ai/v2/docs>.
