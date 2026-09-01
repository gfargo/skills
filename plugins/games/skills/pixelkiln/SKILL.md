---
name: pixelkiln
description: Use PixelKiln to plan, generate, review, recover, audit, pack, and export manifest-driven pixel-art projects. Apply when a task uses the PixelKiln CLI, manifest, lockfile, or generated asset workflow; do not use for unrelated one-off image generation.
---

# PixelKiln

Treat generated pixel art as build state: declared in a manifest, costed before
submission, reviewed by a human, and recorded with exact provenance.

## Working rules

- Locate `pixelkiln.manifest.json` first. Paths are manifest-relative.
- Never inspect, print, or commit provider credentials. PixelKiln can load
  `PIXELLAB_API_KEY` from `.env.local` beside the manifest or from the current
  working directory.
- Run `pixelkiln doctor --dry-run` and `pixelkiln plan` before paid work. Report
  actionable, recoverable, and estimated cost figures with their provider unit.
- Do not regenerate recoverable work. Use `pixelkiln restore` first.
- Submit paid work only when the user has authorized generation. Always pass an
  explicit `--budget` no higher than the authorized estimate.
- Leave visual selection to the local `pixelkiln pick` review page unless the
  user explicitly provides a selection rule. Closing it applies nothing.
- Preserve manual edits and ownership errors. Inspect the difference before any
  `--force` operation.
- Commit the manifest, lockfile, generated outputs, and derived artifact
  companions. Never commit `.env.local` or `.pixelkiln/` cache data.

## Choose the smallest workflow

For ordinary work, prefer:

```bash
pixelkiln doctor --dry-run
pixelkiln plan
pixelkiln gen --budget <approved-provider-units>
pixelkiln audit --check
```

Use the staged `submit` → `poll` → `pick` → `fetch` commands when resuming or
debugging one phase. Use `restore` for missing bytes, `adopt` for exact matches
already in the provider account, and `salvage` for reviewed unclaimed objects.
Use `pack`, `mount`, or `export` only for the artifact format the project needs.

PixelKiln's orchestration is provider-neutral. PixelLab is currently its only
production and live-tested adapter; `FakeProvider` is the deterministic test
adapter. Do not imply that other production providers already work.

When working in the PixelKiln repository, consult `docs/GETTING_STARTED.md` for
the full workflow, `docs/CLI.md` for flags, `docs/MANIFEST.md` for the schema,
and `docs/RECOVERY.md` before account adoption, salvage, discard, or purge.
