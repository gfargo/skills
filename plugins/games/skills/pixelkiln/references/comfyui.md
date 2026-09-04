# ComfyUI

Read this reference before operating a PixelKiln manifest whose provider is
`comfyui`.

## Position it honestly

The ComfyUI adapter automates a committed graph, candidate review, provenance,
and recovery. It does not make a general image model produce good pixel art.
Use it when local control, private inputs, or a custom graph justify manual art
direction. Do not call the current SDXL plus Pixel Art XL setup production-ready
or game-ready.

If the user mainly wants the shortest path to usable pixel art, compare the
specialized hosted providers first. Choose ComfyUI when owning the workflow is
worth the extra model testing and cleanup.

## Minimal safe path

1. Generate two to four candidates on the model's normal working canvas.
2. Reject weak composition and missing prompt elements before post-processing.
3. For isolated art, remove the background at full resolution.
4. Run `pixelkiln refine` to recover one native pixel per detected cell and
   apply the project's final palette, usually 16–32 colors without dithering.
5. Review the native file at 1× and an integer zoom. Check silhouette, clusters,
   contours, single-pixel noise, palette separation, alpha, and seams.
6. Record the named review with `pixelkiln refine approve`, then require
   `pixelkiln refine check` before packaging.
7. Accept the smallest clear result. For the tested stack, start with 48–128px
   native components and compose larger scenes from reviewed parts.

Background removal stays in the ComfyUI graph. PixelKiln automates grid
recovery, final palette enforcement, measurable audits, and the approval
record. It does not decide whether the art is good. Do not present the generated
working canvas, a nearest-neighbor resize, an unreviewed grid recovery, or a
pending quality record as the finished asset.

Test a workflow on at least two different scene families before recommending
it. A prompt that reduced noise in the alpine benchmark increased noise in the
volcanic benchmark and still missed the requested fortress.

## Connection and workflow contract

ComfyUI is self-hosted and needs no provider credential. Never assume the
server is reachable. `pixelkiln doctor --dry-run` validates workflow bindings
offline; `pixelkiln doctor` checks the read-only system-stats route. The default
URL is `http://127.0.0.1:8188`; `COMFYUI_BASE_URL` overrides it. Keep an
unauthenticated server off public networks.

The style's `providerOptions.comfyui` must name a committed API-format workflow,
one image output node, the expected candidate count, and bindings for prompt,
width, height, and batch size. A seed binding is required when the style
declares a seed. Node IDs and model filenames are workflow-specific. Check the
exported JSON instead of guessing them.

Planning parses and hashes the workflow file without contacting ComfyUI. A
workflow content change makes affected assets stale. The adapter supports only
the `map` generator, PNG output from one node, 1–16 candidates, and dimensions
from 16–4096px. Manifest `styleImages` and `palette` are unsupported; those
controls belong in the workflow.

## Tested evidence, not a preset

The diagnostic stack uses SDXL Base 1.0 as
`sd_xl_base_1.0.safetensors` and Pixel Art XL as
`pixel-art-xl.safetensors`. The repository's
`benchmarks/provider-environments/comfyui/` project names both files and uses
core nodes only. Its workflows sample at 1024×1024, then use `ImageScale` with
`nearest-exact`; width and height bindings point to the scale node so the saved
PNG still matches the manifest dimensions. The isolated workflow uses LoRA
strength 1.0, while the environment workflow uses 0.85. Read
`docs/COMFYUI.md` for download links, checksums, licenses, and measured limits.

The cleanup experiment in
`benchmarks/provider-postprocessing/comfyui/` runs BiRefNet on the decoded
full-resolution image, quantizes RGB separately, joins the inverted background
mask as alpha, and scales last. It proves those nodes run; it is not the complete
recommended pipeline. Background removal belongs before grid recovery. Final
palette quantization belongs after grid recovery. Always audit the final native
file, because reconstruction can introduce averaged colors.

The boundary experiment in
`benchmarks/provider-hires/comfyui/` keeps generation canvas, native art grid,
and display size separate. Pixel Art XL can make a 1024px raster whose implied
cells resolve to a much smaller editable grid. Nearest-neighbor scaling
preserves those fake cells; it does not repair them. Run accepted output through
`pixelkiln refine` with the pinned local Retro Diffusion Pixel Art Fixer and an
explicit project palette. Its companion records the fixer revision, source and
output hashes, detected grid, confidence, dimensions, audit, and human-review
state.

Do not add `LatentUpscale` and a second sampling pass by default. The tested
1536px and 2048px variants blurred the output. Nearest-neighbor scaling preserves
existing cells but cannot repair them. Grid recovery proves structure, not good
clusters or prompt coverage.

## Operation and recovery

The plan reports `0 free` and generation uses `--budget 0`. Explain that this
means no metered API charge. Hardware, electricity, hosting, and model licenses
can still cost money.

The lockfile records a prompt ID, output node, workflow hash, and portable
`comfyui://` source. Restore from the validated content cache first. If the
cache is missing, the current ComfyUI server must still retain the history
output so PixelKiln can resolve the portable source.

For the complete setup, composition stack, benchmark, and troubleshooting, use
`docs/COMFYUI.md` or
<https://pixelkiln.griffen.codes/docs/comfyui>.
