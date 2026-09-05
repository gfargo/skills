---
name: pixelkiln
description: Use PixelKiln to plan, generate, review, recover, audit, pack, and export manifest-driven pixel-art projects. Apply when a task uses the PixelKiln CLI, manifest, lockfile, or generated asset workflow; do not use for unrelated one-off image generation.
---

# PixelKiln

Treat generated pixel art as build output. Read the manifest, report the cost
before submission, stop for human review, and record the source and output
hashes.

## Working rules

- Locate `pixelkiln.manifest.json` first. Paths are manifest-relative.
- Never inspect, print, or commit provider credentials. Read the top-level
  provider default and every style override, then load the matching provider
  references below.
- Run `pixelkiln doctor --dry-run` and `pixelkiln plan` before paid work. Report
  actionable, recoverable, and estimated cost figures with their provider unit.
- Do not regenerate recoverable work. Use `pixelkiln restore` first.
- Submit paid work only when the user has authorized generation. Always pass an
  explicit `--budget` no higher than the authorized estimate. For mixed work,
  pass one `--budget provider=amount` ceiling for every paid provider in the plan.
- Leave visual selection to the local `pixelkiln pick` review page unless the
  user explicitly provides a selection rule. Closing it applies nothing.
- When a selected style declares `quality`, run manifest-mode `pixelkiln refine`
  after its source is accepted and downloaded. Read the quality reference below.
  Do not copy manifest-owned palette, path, or threshold settings into flags.
  Never record approval without the named person's completed 1× review.
- Treat every ComfyUI output as source material until it passes the native-grid,
  final-palette, prompt-coverage, and human 1× checks in the ComfyUI reference.
  A successful PNG or high-confidence grid result is not quality approval.
- When the project commits a PixelKiln quality baseline, run `pixelkiln quality
  check --from <baseline>` before packaging. Treat a pass as structural
  continuity, never as human approval or proof that the prompt was satisfied.
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

For a mixed plan, replace the single ceiling with repeated provider-keyed
ceilings copied from each plan group.

Use the staged `submit` → `poll` → `pick` → `fetch` commands when resuming or
debugging one phase. Use `restore` for missing bytes, `adopt` for exact matches
already in the provider account, and `salvage` for reviewed unclaimed objects.
Use `pack`, `mount`, or `export` only for the artifact format the project needs.
Prefer a manifest quality profile when a whole style shares the final-art rule.
Use `refine --from` only for a one-off candidate outside that contract, after
composition review and, for an isolated asset, background removal.

## Provider routing

Read only the reference needed for the current decision:

- For PixelLab configuration, generators, costs, alpha behavior, or account
  operations, read [references/pixellab.md](references/pixellab.md).
- For Retro Diffusion styles, USD budgets, environment assets, animation, or
  experimental limits, read
  [references/retro-diffusion.md](references/retro-diffusion.md).
- For a self-hosted ComfyUI workflow, node bindings, local cost semantics, or
  portable output recovery, read [references/comfyui.md](references/comfyui.md).
- For Scenario models, Compute Unit ceilings, two-part credentials, or durable
  hosted-asset recovery, read [references/scenario.md](references/scenario.md).
- When a bundled or installed recipe can supply the workflow, model hashes, or
  quality contract, read [references/recipes.md](references/recipes.md).
- When a manifest style declares `quality`, or packaging is blocked on derived
  approval, read [references/quality.md](references/quality.md).
- When one game or repository needs more than one provider, read
  [references/mixed-providers.md](references/mixed-providers.md).

`FakeProvider` is the deterministic test adapter. Describe Scenario as
experimental: one BFL Flux 2 Dev profile is live-tested, while other model
schemas and broader batches are not. Do not describe Retro Diffusion as production-ready until representative
multi-candidate, tileset, GIF, and spritesheet live smoke tests pass.

When working in the PixelKiln repository, consult `docs/GETTING_STARTED.md` for
the full workflow, the matching provider setup guide, `docs/CLI.md` for flags,
`docs/MANIFEST.md` for the schema, and
`docs/RECOVERY.md` before account adoption, salvage, discard, or purge.
