# Parallax backgrounds

A manifest's `parallax` section names a set's layers, far to near, and
`pixelkiln parallax preview|export` previews it and writes Tiled, Godot and
LDtk files (offline, free). Generating and finishing the pieces is still the
work below. The step-by-step guide is at
https://pixelkiln.griffen.codes/docs/parallax, and the evidence, with pictures
and numbers, at https://pixelkiln.griffen.codes/docs/research-parallax.

## The command

- A layer is `asset` (from the lockfile; add `style` if the asset has
  several), `file` (a finished picture, relative to the manifest) or `sky`
  (flat bands PixelKiln draws; one colour is a flat fill). Set `speed`, and
  `bottom`/`x` to place it on a canvas whose origin is the view's top-left at
  the start. `extend: true` carries a far range's bottom row to the view's
  foot. Layers repeat sideways unless `repeat: false`; `repeatY` loops up.
- Anything standing on the ground (trees, props) scrolls at the ground's
  speed, usually `1`; on a slower layer it slides across the floor.
- A set has `view` and `travel`; a climbing set adds `climb` and `speedY`.
- `pixelkiln parallax preview <set>` writes a scrolling HTML page and a strip
  of frames. Read its warnings: a rough join (set `"loop": "cut"`), or a
  backdrop that ends before the camera does, sideways or at the top of a
  climb. Export adds one for LDtk tiles past the level's edge.
- `pixelkiln parallax export <set> [--format tiled|godot|ldtk]`. Godot,
  Tiled and Phaser (`setScrollFactor`) were checked pixel for pixel against
  the preview, and LDtk by its drawing code. Tell the user to turn on Godot's
  Rendering > 2D > Snap 2D Transforms to Pixel, and to keep a Tiled map's
  parallax origin at (0, 0), since Tiled 1.11+ applies a non-zero one backwards.
- A `run` layer lays silhouettes (`{ "asset" }` or `{ "file" }` each) in an
  overlapping row by `seed`, with `step`, `gap` and `jitter`. It loops by
  default; `"repeat": false` lays it once across what the camera sees.
- `"foot": 0.3` trims a picture (each sprite of a run) and dithers its bottom
  30% out. `"tint": { "color", "amount" }` mixes it toward a colour and snaps
  it to the set's `palette` (a style id or colours; without one, to the
  picture's own colours). Two runs of the same sprites with different tints
  make two tiers.
- `"loop": "cut"` makes a whole picture join smoothly before `foot` and
  `tint`: it crops to the two columns that best continue into each other.
  Prefer it to `crossfade` (rougher on most layers) and `mirror` (doubles the
  width and reads as mirrored). Not on sky, a run or `repeat: false`.

## The route to try first

Compose each layer in the engine from separate silhouettes rather than cutting
layers out of one scene. Generate a few landforms per biome, one at a time,
each prompted as a lone sprite (see "Generating silhouettes" below), and dither
the foot out afterwards. In the game, lay them in overlapping
runs (each next one about 72% of the way across the last, random flips, never
the same twice in a row, a few pixels of jitter), in two tiers tinted toward a
dusk or haze colour, over a sky the engine draws. A `run` layer in the
manifest does the laying. Nothing hidden has to be
invented, and a run can be as wide as the level. A game using PixelKiln ships
this; its numbers are in step 12 of the research note.

- Small, close rates for the tiers (0.85 and 0.92 of the camera) keep them
  together; wide spreads open bands of sky between them.
- Anchor the layers where they sit with parallax off and let them only rise,
  and fill the strip a rising layer uncovers with a thin band of distant
  ground, not sky, so nothing floats.
- A bounded level needs no loop: extend each run past both ends by the most
  that layer can drift.

## Generating silhouettes

- Describe one object as a sprite, never as distant scenery: "a lone landform
  game sprite on a plain empty background, nothing else in the picture: no
  clouds, no moon, no sun, no hills or ground around it", "about two thirds as
  wide as the picture". Words like "distant", "hazy" or "fading at its foot"
  make pixflux paint a whole scene, and the shape runs off the canvas.
- Leave out a glow at a summit; pixflux draws it as a setting sun.
- Pixflux at 256×128 with background removal, one forced palette, two or three
  seeds an asset. Reject any candidate that touches a side of the canvas.
- Add the distance afterwards with `foot: 0.3`, then a `tint` per tier
  toward its own palette colour, the far tier lighter, the near tier darker.
  One light haze colour for every tier makes depth read backwards.

## Generating whole layers

- PixelLab pixflux with ONE forced palette (`palette`) on every layer is the
  cheapest route that gives layers which agree. Layers with background
  removal, at most 40,000 px each (400×100) to stay transparent. pixflux has
  no seamless option, so give each layer `"loop": "cut"`.
- Draw the sky as flat palette bands instead of generating it: a generated sky
  comes back with mountains in it. A far range comes back as one flat colour;
  recolour it to a palette colour between the sky bands and extend it down to
  the foot of the view so nothing floats. A middle layer that is one landmark
  (a single volcano) goes on screen once, not looped.
- Retro Diffusion's `rd_plus__environment` draws a whole scene whatever the
  prompt says, so it is not a layer generator. Its `tileX` does wrap the
  drawing, but `removeBg` breaks the seam it made.
- For a Retro Diffusion layer that loops sideways, use `rd_plus__default` with
  `tileX`, `removeBg: false` and "a plain flat chroma-key green background" in
  the prompt, then key the background out by script (`parallax.py key`). The
  seam survives (ratios 0.3 to 1.4, against 3.9 with `removeBg`). It drew a
  flat background on 2 of 3 tries, so check before keying.

## A second time of day

- Recolour the finished set onto the new time's palette by luminance rank
  (`parallax.py recolour ... cool`) instead of generating it again. It is free,
  and every shape stays where it was, so the two times line up.
- Take the new palette from the same prompt and seed as the first one, with the
  time of day changed. For night, leave its warm light colours (windows, lamps,
  the moon) out of the swap, or pink-lit snow turns orange.
- Generating again on the new palette keeps the layers consistent with each
  other but moves every peak, and "at night" brought a moon or stars into 7 of
  9 pictures, two of them despite "no moon, no stars" in the prompt.
- Pixel-art background removal is all or nothing per pixel: a silhouette asked
  to fade at its foot comes back with a hard base. Plan a dithered fade, a
  ground strip or a treeline in front.

## Scrolling vertically

- A climb needs tall layers or things placed at height: a tall landmark,
  clouds scattered up the climb, a wall. 400×100 strips run out within a
  quarter of a climb.
- Draw every layer the view's height plus climb × its vertical speed tall, and
  place the layers for the lowest view so a climb never uncovers their feet.
- Scroll the sky's bands vertically (about 0.3) if it should change with
  height; at speed 0 it never does.
- Clouds: prompt puffy cumulus as a lone sprite. Thin streaks and flat banks
  fail in pixflux.
- A wall: pixflux draws a cliff as a pillar with a summit. Keep the summit and
  repeat the body (`parallax.py wall`).
- An endless vertical loop: Retro Diffusion `tileY` on `rd_plus__default`, no
  background removal. Keep the rest of the set on that picture's colours.
- Engines take the y of the same setting: Tiled `parallaxy`, Godot
  `scroll_scale.y`, Phaser `setScrollFactor(x, y)`, LDtk `parallaxFactorY` as
  1 − speed.

## Before generating

- A parallax background is several layers, not one scene. A single scenic
  picture has a focal point, and a repeating focal point is obvious. Put
  landmarks on a layer that does not repeat, and keep looping layers free of
  any single landmark.
- No benchmark background wraps horizontally as generated. Ask for seamless
  output where the provider has it (Retro Diffusion `tileX`), or loop the
  result afterwards and check the join.
- Keep the layers on one palette, one width and one horizon row. Far layers
  should sit closer to the sky colour and lower in contrast. PixelLab's scenes
  already do this; Retro Diffusion's and ComfyUI's benchmark scenes did not.

## Splitting a picture you already have

- A monocular depth model (Depth Anything V2 Small, Apache-2.0, about a second
  on a CPU) gives usable bands for an open landscape. Colour alone does not.
- It fails on a scene framed by tall near objects, and anything floating
  (smoke, clouds, a moon) must be cut out as its own sprite.
- Cut per pixel, not per column, so floating things keep their own layer.
- Whatever a near layer hid has to be invented when it moves. Say so before
  promising a clean result from a flat picture. PixelLab `inpaint_image` on
  the invented pixels gives a believable fill and leaves the rest untouched,
  at 10 to 25 generations a layer by canvas size.
- Do not reach for a layer-splitting model (Qwen-Image-Layered) for depth
  bands: it separated sky from everything else, kept near and far land in
  one layer, and took over half an hour a picture on a 12 GB card.
- For ground that recedes to a horizon, per-row (line) scrolling of the one
  picture needs no cutting at all, but it shears tall shapes.

## In the game

- Round the camera to whole pixels first, then derive each layer's offset from
  that. Rounding each layer separately moves layers while the camera is still.
- Factors of 1/2, 1/4 and 1/8 step on an even rhythm; 0.3 or 0.6 do not.
- Engine settings for a layer at a quarter of the camera's speed: Tiled
  `parallaxx: 0.25`; Godot `Parallax2D.scroll_scale = (0.25, 1)`; Phaser
  `setScrollFactor(0.25, 1)`; LDtk `parallaxFactorX: 0.75` with
  `parallaxScaling: false`. LDtk's factor is how much the layer follows the
  camera, and with scaling on it also shrinks the layer off the pixel grid.
  `pixelkiln parallax export` writes these for you.
