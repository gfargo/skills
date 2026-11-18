# 3D models: texture fixes, the guide, and Blender

A model is a `.glb` file anywhere under a style's `outDir`. It is not a
manifest asset: it has no plan state, lockfile entry, cost, or `gen` step, and
nothing about it is written to the manifest or the lockfile. `pixelkiln model`
works on these files; the gallery shows the same ones in its **3D models**
section.

## The rule

The model as built is never written. Every change goes to an edited copy
beside it under the style's `edits/` folder:

```text
art/characters/models/knight.glb                     the model as built
art/characters/edits/models/knight.glb               the edited copy
art/characters/edits/models/knight.edit.json         what the copy is based on, per texture
art/characters/edits/models/knight.image-<n>.png     a texture exported for painting
art/characters/edits/models/knight.image-<n>.guide.png
art/characters/edits/models/knight.image-<n>.guide.txt
art/characters/edits/models/knight.image-<n>.pxo     layers, when painted in the gallery
art/characters/edits/models/knight.export.glb        what Blender (or another tool) exported
```

A consumer that wants the fixed model takes the edited copy when it exists.
`pack`, `mount`, and `export` do not read models.

## Find what there is

```bash
pixelkiln model list            # text: meshes, bones, animations, every image and what it is for
pixelkiln model list --json     # the same, plus the edit's status and record
```

Each image is listed by index with its material and role (`base colour`,
`normal map`). Name a texture by index (`--image 8`) or by material name
(`--image Helm`, its base colour texture). Name a model by its path, its file
name (`knight`), or the gallery id (`model:cast/models/knight.glb`).

An edit's `status` is `edited` (based on the model as it is now),
`rebuilt-since` (the model was rebuilt after the edit; tell the person before
building on it), or `unrecorded` (a file placed by hand, no record).

## Fix a texture by hand

```bash
pixelkiln model export knight --image Helm --no-open
# the person paints edits/models/knight.image-8.png
pixelkiln model import knight --image Helm
```

`export` writes the texture as PNG (decoding a JPEG one), starting from the
edited copy when there is one so earlier fixes carry on, plus the guide:

- `.guide.png`, transparent and the texture's size: each piece of the
  automatic unwrap outlined in its body region's colour and numbered. The
  person lays it on top as a layer and never paints on it.
- `.guide.txt`: the number, colour, and name of each region ("Left lower
  arm"), then every piece with its label ("Left lower arm, front"), pixel
  bounds, and meshes. Use it to answer "where on the texture is the left
  forearm?" or to point the person at a piece. `--json` on `export` returns
  the paths, piece count, and region names.

Regions come from the mesh: pieces are found from the texture coordinates,
and each is named after the bone with the most skin weight over it, or its
mesh when there is no skeleton. A piece's facing (front, back, left, right,
top, underside) is in the model's own frame, so "left" is the figure's left.

`import` refuses a PNG of a different size from the texture and writes the
copy with only that image replaced; meshes, normals, tangents, skins,
animations and the other images keep their bytes. `--from <png>` imports a
file from elsewhere. Without `--no-open`, `export` opens the texture in
`PIXELKILN_EDITOR` or the OS default, which an agent should not do unless
asked.

Do not paint the texture yourself unless the person asks for a specific,
checkable change. Hand fixes (streaks, pinholes, seams) are judgement calls
the person makes looking at the model. The gallery is better for that: its
editor shows the guide with names written in (on a layer of its own with a
protocol 5 editor build, which `pixelkiln tools status` reports) and the 3D
model beside the canvas, updating as they paint. Suggest
`pixelkiln gallery --edit` and leave `--edit` to them.

## Blender

```bash
pixelkiln model blender knight
# the person paints in Texture Paint, then File > Export > PixelKiln edit (.glb)
pixelkiln model adopt knight                 # changed base colour textures only
pixelkiln model adopt knight --mode whole    # the exported file as the edit
```

`blender` starts the person's Blender on the edited copy (or the model as
built) with an export item that writes `knight.export.glb`. It finds Blender
through `PIXELKILN_BLENDER`, then the usual install location; it never
installs it. It opens a window, so run it only when the person asks.

`adopt` defaults to `textures`: the base colour textures that changed in the
export are matched to the model by material name and put into a copy of the
model as built. Blender re-encodes meshes and animations on export, so this
keeps them exactly as built. Use `--mode whole` only when the person changed
more than paint (geometry, weights, animation). `--from <glb>` adopts a file
from elsewhere. An export whose material names no longer match is refused;
`whole` is the way to take it.

## Not covered

Normal maps and other non-colour images are listed but not offered for
painting; a hand-painted normal map is rarely right. There is no way to
declare the edited copy in the manifest, and no step builds or regenerates a
model.
