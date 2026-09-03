# Mixed-provider projects

Read this reference when one game or repository needs assets from both PixelLab
and Retro Diffusion.

One manifest selects one top-level `provider`. Do not add undocumented
per-style provider fields: the current CLI constructs one account adapter for a
manifest run, and a single `--budget` has one provider-specific unit.

Use one manifest and lockfile per provider inside the same project:

```text
art/
  pixelkiln.pixellab.manifest.json
  pixelkiln.pixellab.lock.json
  pixelkiln.retrodiffusion.manifest.json
  pixelkiln.retrodiffusion.lock.json
pixelkiln.workspace.json
```

Keep their output directories distinct. Plan and authorize them separately:

```bash
pixelkiln plan --manifest art/pixelkiln.pixellab.manifest.json --lock art/pixelkiln.pixellab.lock.json
pixelkiln gen --manifest art/pixelkiln.pixellab.manifest.json --lock art/pixelkiln.pixellab.lock.json --budget <generations>

pixelkiln plan --manifest art/pixelkiln.retrodiffusion.manifest.json --lock art/pixelkiln.retrodiffusion.lock.json
pixelkiln gen --manifest art/pixelkiln.retrodiffusion.manifest.json --lock art/pixelkiln.retrodiffusion.lock.json --budget <usd>
```

Register both manifests in the workspace catalog so aggregate status and claim
checks see the whole project. A mixed-provider warning is expected because it
prevents account-wide commands from silently assuming one backend.

Package each manifest's reviewed outputs independently, or use `pixelkiln pack
--inputs <file> --out <path>` with an explicit JSON list when the final sheet
must combine files from both providers. Never merge the two lockfiles or add
generation counts to USD. Each entry must retain the provider that produced it.

A practical split is PixelLab for prompt-sensitive buildings and mature account
recovery, then Retro Diffusion for environment-styled backdrops, clean cutouts,
or native animation. The committed benchmark is evidence for those tendencies,
not a guarantee; run one representative asset before expanding either batch.
