# Controlled revisions

Read this reference when an asset declares `revision`.

- Treat the parent and child as separate assets. `revision.from` must name a
  parent in the same style.
- Run `pixelkiln plan --only <child>` first. If it reports `blocked`, do not try
  to submit the child or work around the gate.
- A generated parent must be current and downloaded. When the parent has a
  quality profile, its named human approval must be current. Never approve it
  on the user's behalf.
- An inpaint mask must be a PNG and match the available parent dimensions.
- Start image-to-image strength around `0.2`–`0.4`; explain that the exact
  effect belongs to the model and graph.
- A filtered child resolves its parents for safety but does not generate them.
  Generate, fetch, refine, and approve the parent explicitly, then re-plan.
- PixelKiln rechecks parent and mask bytes immediately before submission. A
  changed input must stop the run before provider work begins.
- In candidate review, compare the source beside every output. Reject
  silhouette or layout drift before post-processing.
- ComfyUI is the only built-in revision adapter. Its workflow needs
  `sourceImage`; inpaint also needs `maskImage`; declared strength needs a
  `strength` binding.
- The bundled `comfyui/pixel-art-xl-img2img@1.0.0` recipe is square-only. Its
  live three-strength fortress smoke preserved the footprint but missed the
  requested snow, lost alpha, and produced thousands of colors. Do not call it
  production-ready or a quality preset.

Commit parent inputs, masks, outputs, lock lineage, and quality companions.
Do not commit ComfyUI's recreated input cache.
