# Mixed-provider projects

Read this reference when one game or repository needs assets from PixelLab,
Retro Diffusion, ComfyUI, or another combination of providers.

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
  pixelkiln.comfyui.manifest.json
  pixelkiln.comfyui.lock.json
pixelkiln.workspace.json
```

Keep their output directories distinct. Plan and authorize them separately:

```bash
pixelkiln plan --manifest art/pixelkiln.pixellab.manifest.json --lock art/pixelkiln.pixellab.lock.json
pixelkiln gen --manifest art/pixelkiln.pixellab.manifest.json --lock art/pixelkiln.pixellab.lock.json --budget <generations>

pixelkiln plan --manifest art/pixelkiln.retrodiffusion.manifest.json --lock art/pixelkiln.retrodiffusion.lock.json
pixelkiln gen --manifest art/pixelkiln.retrodiffusion.manifest.json --lock art/pixelkiln.retrodiffusion.lock.json --budget <usd>

pixelkiln plan --manifest art/pixelkiln.comfyui.manifest.json --lock art/pixelkiln.comfyui.lock.json
pixelkiln gen --manifest art/pixelkiln.comfyui.manifest.json --lock art/pixelkiln.comfyui.lock.json --budget 0
```

Register every manifest in the workspace catalog so aggregate status and claim
checks see the whole project. A mixed-provider warning is expected because it
prevents account-wide commands from silently assuming one backend.

Package each manifest's reviewed outputs independently, or use `pixelkiln pack
--inputs <file> --out <path>` with an explicit JSON list when the final sheet
must combine files from multiple providers. Never merge their lockfiles or add
generation counts, USD, and `free` plans. Each entry must retain the provider
that produced it.

A practical split is PixelLab for prompt-sensitive buildings and mature account
recovery, then Retro Diffusion for environment-styled backdrops, clean cutouts,
or native animation. ComfyUI fits private work, local models, and custom graph
control when the user accepts a refinement pass and hands-on art review.
`pixelkiln refine` can apply the same native-grid, palette, audit, and approval
record to output from any provider. Start with a specialized hosted provider when
minimizing cleanup matters more than local control. The committed benchmark is
evidence for tendencies, not a guarantee. Run one representative asset before
expanding any batch, and test a ComfyUI graph on at least two scene families.
