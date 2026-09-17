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

A base can start from the author's own south-facing sprite (`reference` on
the asset; standard wants it at the style's size, v3 up to 256px, pro up
to 168px), and a standard humanoid base takes `proportions` (`chibi`,
`heroic`, or multipliers) on the style or the asset. A pro base can be
designed from a `concept` image (up to 1024px) and follow the look of
another generated character in the style (`styleCharacter`), which makes
that character a dependency the way a loop's parent is.

A v3 loop can start from a pose image (`startFrame`) or interpolate to one
(`endFrame`), take a `subject` when the character's own description would
mislead, and ask PixelLab to `enhancePrompt`; a template loop takes
`outline`, `shading`, and `detail` overrides instead.

A loop costs per direction, and a sprite facing east is the sprite facing
west flipped. Declare the west loop and make the east one `{ "mirror":
"bot.walk.west" }`: PixelKiln flips it locally for nothing, in the wave
after the source lands. South and north cannot be mirrored. Eight directions
of one loop are then 5 generations, not 8.

In the committed environment benchmark, PixelLab followed complex building
prompts more closely and produced broader, more separable depth bands in scenic
backgrounds. Its 256px and 384px map objects were opaque despite the route's
transparency claim, so inspect alpha on one representative result before a
batch. Inspect scenic outputs for stray marks too: one untouched 384px attempt
contained a generated signature-like glyph.

For setup and current field constraints, use
<https://pixelkiln.griffen.codes/docs/pixellab>. When working in the PixelKiln
repository, `docs/PIXELLAB.md` and `docs/ENDPOINTS.md` are the canonical local
sources. The provider sources are <https://www.pixellab.ai/> and
<https://api.pixellab.ai/v2/docs>.
