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

In the committed environment benchmark, PixelLab followed complex building
prompts more closely and produced the stronger scenic background. Its 256px map
objects were opaque despite the route's transparency claim, so inspect alpha on
one representative result before a batch.

For setup and current field constraints, use
<https://pixelkiln.griffen.codes/docs/pixellab>. When working in the PixelKiln
repository, `docs/PIXELLAB.md` and `docs/ENDPOINTS.md` are the canonical local
sources. The provider sources are <https://www.pixellab.ai/> and
<https://api.pixellab.ai/v2/docs>.
