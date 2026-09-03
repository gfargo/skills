# ComfyUI

Read this reference before operating a PixelKiln manifest whose provider is
`comfyui`.

ComfyUI is self-hosted and needs no provider credential. Never assume the
server is running or reachable. `pixelkiln doctor --dry-run` validates the
workflow and bindings offline; `pixelkiln doctor` checks the configured server
through its read-only system-stats route. The default URL is
`http://127.0.0.1:8188`; `COMFYUI_BASE_URL` overrides it. Do not advise exposing
an unauthenticated ComfyUI server to a public network.

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

For the tested higher-quality pixel-art stack, use SDXL Base 1.0 as
`sd_xl_base_1.0.safetensors` and Pixel Art XL as
`pixel-art-xl.safetensors`. The repository's
`benchmarks/provider-environments/comfyui/` project names both files and uses
core nodes only. Its workflows sample at 1024×1024, then use `ImageScale` with
`nearest-exact`; width and height bindings point to the scale node so the saved
PNG still matches the manifest dimensions. The isolated workflow uses LoRA
strength 1.0, while the environment workflow uses 0.85. Read
`docs/COMFYUI.md` for download links, checksums, licenses, and measured limits.

The plan reports `0 free` and generation uses `--budget 0`. Explain that this
means no metered API charge. Hardware, electricity, hosting, and model licenses
can still cost money.

The lockfile records a prompt ID, output node, workflow hash, and portable
`comfyui://` source. Restore from the validated content cache first. If the
cache is missing, the current ComfyUI server must still retain the history
output so PixelKiln can resolve the portable source.

For the complete setup, quality stack, benchmark, and troubleshooting, use
`docs/COMFYUI.md` or
<https://pixelkiln.griffen.codes/docs/comfyui>.
