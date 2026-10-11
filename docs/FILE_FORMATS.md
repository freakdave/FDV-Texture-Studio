# FDV Texture Studio · File formats

[← Back to the user guide](USER_GUIDE.md)

This reference describes the files that FDV Texture Studio exports.<br>
Use it to read Studio exports in another game engine or tool.

| Format                | Extension                      | Section                                                         |
| :-------------------- | :----------------------------- | :-------------------------------------------------------------- |
| FDV material bundle   | `.fdvmat` and a folder         | [FDV material](#1-fdv-material-fdvmat)                          |
| Dreamcast textures    | `.dtex`, `.tex`, `.dt`, `.pvr` | [Dreamcast texture containers](#2-dreamcast-texture-containers) |
| Palette               | `.pal`                         | [Palette](#29-palette-pal)                                      |
| Sprite sheet metadata | `.sprites.json`                | [Sprite sheets](#3-sprite-sheet-metadata-spritesjson)           |
| Studio graph          | `.fdvgraph`                    | [Graphs](#4-studio-graph-fdvgraph)                              |

**Unspecified** in this document means that the Studio does not define the behavior.<br>
Do not depend on an Unspecified value.

## Conventions

- All binary values are **little-endian**. Offsets are in bytes from the start of the file, counting from 0.
- JSON files use UTF-8 with an indent of two spaces. The Studio writes object keys in alphabetical order. Key order has no meaning.
- Paths inside JSON files use forward slashes.
- The Studio writes PNG files with four channels: red, green, blue and alpha, 8 bits each. PNG row 0 is the top of the picture.

### Terms

| Term                | Meaning                                                                                                                                                                          |
| :------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Texel**           | One pixel of a texture.                                                                                                                                                          |
| **PVR**             | The PowerVR2 graphics chip in the Sega Dreamcast. The Dreamcast containers store data in the layout that this chip reads directly.                                               |
| **Mip chain**       | The texture plus a series of smaller copies. Each copy has half the width and height of the previous one, down to 1 × 1. The chip selects a copy by on-screen size.              |
| **Twiddled**        | A texel order in which the bits of the x and y coordinates are interleaved (Morton or Z order). The PVR reads twiddled textures faster. The opposite is **linear** (row by row). |
| **Stride**          | A linear texture whose width is not a power of two. Its width is a multiple of 32.                                                                                               |
| **VQ**              | Vector quantization, a compression method. The file stores a table of 8-byte blocks (the **codebook**) and one index byte per block of texels.                                   |
| **Palette**         | A list of colors. A paletted texture stores a 4-bit or 8-bit color index for each texel.                                                                                       |
| **Environment map** | A second texture that FDV draws over a surface as a reflection or highlight.                                                                                                     |

---

## 1. FDV material (`.fdvmat`)

### 1.1 What it is

An `.fdvmat` file is a JSON object.<br>
It names one surface texture and the settings that FDV uses to draw that surface on a mesh.<br>
**Export FDV material…** writes it together with the textures that it references.

FDV Editor imports `.fdvmat` files into its material library.<br>
FDV Engine does not read `.fdvmat` files at run time. FDV Editor converts each material into the engine's mesh and level formats when it exports a game level.

### 1.2 Bundle folder layout

The Studio creates one folder for each export. The folder name is the material name.<br>
In this table, `<name>` is the material name.

| File                      | When it is written                                                                 | Contents                                                          |
| :------------------------ | :--------------------------------------------------------------------------------- | :---------------------------------------------------------------- |
| `<name>.fdvmat`           | Always                                                                             | The material.                                                     |
| `<name>.png`              | Always                                                                             | The surface texture, largest mip level only.                      |
| `<name>.dtex`             | **Include Dreamcast DTEX** is on                                                   | The surface texture as a DTEX file. See [DTEX](#26-dtex-and-tex). |
| `environment.png`         | An environment image is selected                                                   | The environment map after the Studio resizes and filters it.      |
| `environment.dtex`        | DTEX is on, an environment image is selected, and the environment mode is not None | The processed environment map as DTEX.                            |
| `sources/environment.png` | An environment image is selected                                                   | The original, full-size environment image.                        |
| `<name>.fdvgraph`         | The material came from a graph                                                     | The editable graph. See [Graphs](#4-studio-graph-fdvgraph).       |
| `<name>.obj`, `<name>.mtl` | **Include OBJ/MTL preview** is on                                                 | A preview quad and material. See [OBJ/MTL preview](#5-objmtl-preview). |
| `sources/input.png`       | The graph uses an imported image                                                   | A copy of that image.                                             |

- If the material name, ignoring letter case, is `environment`, the environment files are named `reflection.png` and `reflection.dtex`. The files in `sources/` keep their names.
- The `.fdvmat` references only the surface texture and the environment map. The `sources/` folder and the graph are for later editing in the Studio.
- The Studio writes the bundle into a temporary folder and renames it when every file is complete. It never replaces an existing bundle.

**Material names** contain 1 to 26 UTF-8 bytes.<br>
They cannot contain control characters or `< > : " / \ | ? *`. They cannot start with a dot or end with a dot or a space. Windows device names such as `CON` and `COM1` are refused.<br>
The 26-byte limit keeps `<name>.dtex` within the 31 bytes that FDV runtime file names allow.

### 1.3 Texture references and formats

The `texture` and `envMap` keys hold a file name with its extension. There is no separate format key.<br>
In a Studio export, `assetPathsRelative` is `true` and each reference is relative to the folder that contains the `.fdvmat`.

| Key                                        | DTEX off          | DTEX on            |
| :----------------------------------------- | :---------------- | :----------------- |
| `texture`                                  | `<name>.png`      | `<name>.dtex`      |
| `envMap`                                   | `environment.png` | `environment.dtex` |
| `envMap` when the environment mode is None | `""`              | `""`               |

The DTEX bundle also contains the PNG files. A PNG and a DTEX that share a base name show the same picture.

The JSON format does not depend on a texture format. Each reader selects a decoder by the extension of the referenced file. The FDV tools support different extensions:

| Format               | Studio material export writes | FDV Engine, Dreamcast | FDV Engine, desktop | FDV Editor, preview | FDV Editor, import |
| :------------------- | :---------------------------: | :-------------------: | :-----------------: | :-----------------: | :----------------: |
| PNG `.png`           |              Yes              |        No (1)         |         Yes         |         Yes         |      Yes (4)       |
| JPEG `.jpg`, `.jpeg` |              No               |        No (1)         |         Yes         |         Yes         |      Yes (4)       |
| BMP, GIF             |              No               |        No (1)         |         Yes         |         Yes         |      Yes (4)       |
| TGA, PSD, HDR, PNM   |              No               |        No (1)         |         No          |         Yes         |      Yes (4)       |
| DTEX `.dtex`         |              Yes              |        Yes (2)        |     Via PNG (3)     |       Yes (5)       |      Yes (4)       |
| TEX `.tex`           |              No               |        No (1)         |         No          |         No          |      Yes (4)       |
| DT `.dt`             |              No               |        No (1)         |         No          |         No          |      Yes (4)       |
| PVR `.pvr`           |              No               |        No (1)         |       No (6)        |         No          |      Yes (4)       |

Notes:

1. On Dreamcast, FDV Engine always replaces the extension with `.dtex` and loads that file. A reference to `wall.png` loads `wall.dtex`.
2. FDV Engine on Dreamcast accepts ARGB1555, RGB565 and ARGB4444 DTEX files with power-of-two sizes from 8 to 1024. It accepts twiddled or linear data and VQ with or without mipmaps. It **rejects** uncompressed DTEX files that have mipmaps, and it rejects YUV422, normal-map and paletted DTEX files.
3. FDV Engine on desktop replaces `.dtex` with `.png` and loads the PNG. It compares extensions with letter case, so `.PNG` fails.
4. Import copies the referenced file. It also copies files that share its base name (for example `<name>.png` beside `<name>.dtex`). It does not decode the file.
5. The FDV Editor preview decodes ARGB1555, RGB565 and ARGB4444 DTEX files. It does not decode stride, YUV422, normal-map or paletted files.
6. FDV Engine on desktop recognizes `.pvr` but does not load it.

A `.pal` file is a palette, not a texture. The `.fdvmat` format has no key for a palette, and no FDV reader accepts a paletted texture in a material.

When FDV Editor exports a level for Dreamcast, it ships the `.dtex` file that has the same base name as the reference. Include Dreamcast DTEX when the material will run on a Dreamcast.

**Rule for other engines**

1. If `assetPathsRelative` is `true`, resolve `texture` and `envMap` relative to the folder that contains the `.fdvmat`. Refuse absolute paths.
2. An empty string means no texture.
3. Select the decoder by the first four bytes, then by the extension: `DTEX` for DTEX or TEX, `DcTx` for DT, `PVRT` or `GBIX` for PVR, otherwise a normal image.
4. If you cannot decode the referenced file, try the file with the same base name and the `.png` extension. Studio bundles always contain it.
5. Use the orientation rule in [Orientation](#210-orientation). The DTEX in a bundle is stored upside down relative to the PNG.

### 1.4 Keys

The Studio writes every key in this section in every `.fdvmat`.<br>
"Default" is the value when the user does not change the setting. "Range" is the range that the Studio accepts.

#### Identity and files

| Key                  | Type    | Default | Meaning                                                                                                 |
| :------------------- | :------ | :------ | :------------------------------------------------------------------------------------------------------ |
| `version`            | integer | `1`     | `2` if `wrapU` or `wrapV` is not 0, otherwise `1`. See [Compatibility](#15-versions-and-compatibility). |
| `name`               | string  | none    | The material name. FDV uses the name as the identity of the material.                                   |
| `assetPathsRelative` | boolean | `true`  | `true`: `texture` and `envMap` are relative to the `.fdvmat` folder.                                    |
| `texture`            | string  | none    | The surface texture file. See [1.3](#13-texture-references-and-formats).                                |
| `studioGraph`        | string  | `""`    | `<name>.fdvgraph` when the bundle contains a graph, otherwise `""`. Information only.                   |

#### Surface

| Key                  | Type               | Default        | Range       | Meaning                                                                                                                                                                                                            |
| :------------------- | :----------------- | :------------- | :---------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `color`              | array of 4 numbers | `[1, 1, 1, 1]` | 0 to 1 each | Red, green, blue and alpha. FDV multiplies the texture by this color.                                                                                                                                             |
| `lighting`           | integer            | `0`            | 0 or 1      | `0` Lit: scene and baked lights apply. `1` Unlit: full texture color, no lighting. FDV also turns fog off for an Unlit material.                                                                                  |
| `fog`                | boolean            | `true`         | none        | The surface receives distance fog.                                                                                                                                                                                 |
| `twoSided`           | boolean            | `false`        | none        | `true` draws both faces. `false` removes back faces.                                                                                                                                                               |
| `depthTest`          | boolean            | `true`         | none        | `false`: the surface ignores the depth buffer and draws over earlier geometry. There is no depth-write key. On the PVR, opaque and cutout surfaces write depth and blended surfaces do not.                        |
| `transparency`       | integer            | `0`            | 0 to 2      | `0` Opaque. `1` Cutout: texels are fully drawn or discarded (PVR punch-through list). `2` Blended: mixed with the background by `blend` (PVR translucent list).                                                    |
| `blend`              | integer            | `4`            | 1 to 5      | The mix for a Blended surface. FDV reads it only when `transparency` is `2`. See [Blend modes](#blend-modes).                                                                                                      |
| `alphaTestThreshold` | number             | `0.25`         | 0 to 1      | The cutout limit for a Cutout surface. FDV on desktop draws a texel only when its alpha is greater than this value.                                                                                                |
| `alphaHoles`         | integer            | `0`            | 0 or 1      | The meaning of transparent texels. `0` See-through: holes are cut out. `1` Reflect: the surface stays opaque, holes draw in their own color, and the environment map fills them. `1` requires `transparency` `0`. |
| `surface`            | string             | `""`           | see below   | The driving surface for vehicle physics. `""` means not set.                                                                                                                                                       |

`surface` values: `tarmac`, `concrete`, `gravel`, `dirt`, `mud`, `sand`, `snow`, `ice`, `cobble`, `grass`, `rock`.<br>
FDV Editor reads an unknown name as not set.

#### Sampling

| Key              | Type    | Default | Range   | Meaning                                                                                                                                                                                                                             |
| :--------------- | :------ | :------ | :------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `filter`         | integer | `0`     | 0 or 1  | `0` bilinear (smooth). `1` nearest (sharp texels).                                                                                                                                                                                  |
| `wrapU`, `wrapV` | integer | `0`     | 0 to 2  | Behavior outside the 0 to 1 texture range. `0` Repeat. `1` Clamp. `2` Mirrored repeat.                                                                                                                                             |
| `useMipmaps`     | boolean | `true`  | none    | Use the mip chain of the texture file when it has one. The Studio copies the **Mipmaps** export option.                                                                                                                             |
| `mipBias`        | integer | `2`     | 1 to 15 | The PVR mip bias, in steps of 0.25. Bias = value × 0.25. `4` (1.00) is neutral. Lower values select larger mip levels (sharper). Higher values select smaller mip levels (softer, less texture bandwidth). The default `2` is 0.50. |

#### Tiling and scrolling

| Key                  | Type               | Default  | Range   | Meaning                                                                                                                                             |
| :------------------- | :----------------- | :------- | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tilingU`, `tilingV` | number             | `1`      | 0 to 64 | Repeats per meter: copies of the texture per meter of surface along U and V. `0` means not stated. FDV Editor uses these values to set UV mapping.  |
| `uvTiling`           | array of 2 numbers | `[1, 1]` | none    | A UV scale that FDV Editor applies when it generates some geometry. The Studio always writes `[1, 1]`.                                              |
| `uvOffset`           | array of 2 numbers | `[0, 0]` | none    | A UV offset that FDV Editor applies when it generates some geometry. The Studio always writes `[0, 0]`.                                             |
| `uvScroll`           | array of 2 numbers | `[0, 0]` | finite  | Texture scroll speed in U and V, in wraps per second. A wrap is one full texture width or height. The sign convention for direction is Unspecified. |

#### Environment map

| Key                        | Type               | Default        | Range       | Meaning                                                                                                                            |
| :------------------------- | :----------------- | :------------- | :---------- | :--------------------------------------------------------------------------------------------------------------------------------- |
| `envMapMode`               | integer            | `0`            | 0 to 3      | `0` None. `1` Reflection. `2` Gloss. `3` Alpha mask. See below.                                                                    |
| `envMap`                   | string             | `""`           | none        | The environment map file. `""` when `envMapMode` is `0`.                                                                           |
| `envMapBlend`              | integer            | `0`            | 0 to 8      | How the reflection pass mixes with the surface. FDV reads it only in mode `1`. See [Blend modes](#blend-modes).                    |
| `envMipBias`               | integer            | `0`            | 0 to 15     | Mip bias of the environment map, on the `mipBias` scale. `0` uses `mipBias`.                                                       |
| `envTint`                  | array of 4 numbers | `[1, 1, 1, 1]` | 0 to 1 each | RGBA multiplier for the environment map. Alpha controls the strength of the pass.                                                  |
| `envUvScale`               | array of 2 numbers | `[1, 1]`       | 0.01 to 64  | Repeats of the environment map across the surface.                                                                                 |
| `envRotDamp`               | number             | `1`            | 0 to 1      | Camera influence. `1` moves like a mirror. `0` keeps the reflection fixed. Ignored when `envFreezeRotation` is `true`.             |
| `envFreezeRotation`        | boolean            | `false`        | none        | Compute the reflection from fixed world axes and ignore the camera.                                                                |
| `envMirror`                | boolean            | `false`        | none        | Use the view reflection vector instead of the surface normal. A flat surface then reflects a varied picture instead of one color. |
| `envBendDeg`               | number             | `0`            | 0 to 180    | Normal bend in degrees. Tilts vertex normals toward the face edges, for the reflection only. `0` is off.                           |
| `envScrollU`, `envScrollV` | number             | `0`            | finite      | Environment map scroll speed in wraps per second. Independent of `uvScroll`.                                                       |
| `envAlphaMaskCutout`       | boolean            | `false`        | none        | Mode `3` only. `true`: the pass uses alpha blending, so alpha 0 shows the surface. `false`: an additive sheen.                     |

Environment modes:

| Value | Mode       | Pass mix                                                                                     |
| :---- | :--------- | :------------------------------------------------------------------------------------------- |
| `0`   | None       | No pass.                                                                                     |
| `1`   | Reflection | `envMapBlend`.                                                                               |
| `2`   | Gloss      | Always additive (source alpha, one).                                                         |
| `3`   | Alpha mask | Source alpha and inverse source alpha if `envAlphaMaskCutout` is `true`, otherwise additive. |

Note: FDV Editor and FDV Engine store `envBendDeg` in one byte. Values from 0 to 60 keep steps of 0.25°. Values above 60 use steps of 8°.

#### Blend modes

A blend mode selects two factors. The result is `source × source factor + destination × destination factor`.<br>
The **source** is the new texel. The **destination** is the color already in the frame.

| Value | Name                           | Source factor         | Destination factor |    `blend`    | `envMapBlend` |
| :---- | :----------------------------- | :-------------------- | :----------------- | :-----------: | :-----------: |
| `0`   | Engine default                 | source alpha          | 1 − source alpha   |       none         | Yes      |
| `1`   | Additive                       | source alpha          | 1                  |      Yes      |      Yes      |
| `2`   | Multiply                       | 0                     | destination color |      Yes      |      Yes      |
| `3`   | Premultiplied alpha            | 1                     | 1 − source alpha   |      Yes      |      Yes      |
| `4`   | Alpha blend                    | source alpha          | 1 − source alpha   | Yes (default) |      Yes      |
| `5`   | Alpha blend, destination alpha | source alpha          | destination alpha  |      Yes      |      Yes      |
| `6`   | Replace                        | 1                     | 0                  |       none         | Yes      |
| `7`   | Holes reflect (add)            | 1 − destination alpha | 1                  |       none         | Yes      |
| `8`   | Holes mirror (replace)         | 1 − destination alpha | destination alpha  |       none         | Yes      |

- Modes `7` and `8` read the alpha that the opaque surface pass left in the frame. Alpha 0 marks a hole. They work only with `envMapMode` `1` and `alphaHoles` `1`. In mode `1`, the Studio refuses them unless `alphaHoles` is `1`.
- When DTEX is on, the Studio refuses RGB565 for a material whose `transparency` or `alphaHoles` is not `0`, because RGB565 has no alpha.

### 1.5 Versions and compatibility

- `version` is `1` or `2`. Version `2` means that `wrapU` or `wrapV` is not 0. The Studio writes both wrap keys in both versions.
- Neither FDV Editor nor the Studio checks `version` when it reads a file.
- Readers ignore unknown keys.
- A missing key takes the default in the key tables, with one exception: FDV Editor reads a missing `tilingU` or `tilingV` as `0` (not stated).
- FDV Editor clamps out-of-range values to the ranges above. It refuses a file whose `wrapU` or `wrapV` is not 0, 1 or 2. The Studio refuses any out-of-range value.
- `.fdvmat` files that FDV Editor saves can also contain `importedAsset` and `tilingInherited`. Both are editor bookkeeping and do not change drawing. FDV Editor writes `wrapU` and `wrapV` only when one of them is not 0.

### 1.6 Example

A Dreamcast bundle for a material named `Moss`, with default settings and the `grass` surface:

```json
{
  "alphaHoles": 0,
  "alphaTestThreshold": 0.25,
  "assetPathsRelative": true,
  "blend": 4,
  "color": [1.0, 1.0, 1.0, 1.0],
  "depthTest": true,
  "envAlphaMaskCutout": false,
  "envBendDeg": 0.0,
  "envFreezeRotation": false,
  "envMap": "",
  "envMapBlend": 0,
  "envMapMode": 0,
  "envMipBias": 0,
  "envMirror": false,
  "envRotDamp": 1.0,
  "envScrollU": 0.0,
  "envScrollV": 0.0,
  "envTint": [1.0, 1.0, 1.0, 1.0],
  "envUvScale": [1.0, 1.0],
  "filter": 0,
  "fog": true,
  "lighting": 0,
  "mipBias": 2,
  "name": "Moss",
  "studioGraph": "Moss.fdvgraph",
  "surface": "grass",
  "texture": "Moss.dtex",
  "tilingU": 1.0,
  "tilingV": 1.0,
  "transparency": 0,
  "twoSided": false,
  "useMipmaps": true,
  "uvOffset": [0.0, 0.0],
  "uvScroll": [0.0, 0.0],
  "uvTiling": [1.0, 1.0],
  "version": 1,
  "wrapU": 0,
  "wrapV": 0
}
```

The folder `Moss/` also contains `Moss.png`, `Moss.dtex` and `Moss.fdvgraph`.<br>
The Studio writes each array element on its own line. The example shows arrays on one line to save space.

---

## 2. Dreamcast texture containers

### 2.1 Overview

The Studio uses the bundled **pvrtex** encoder to write Dreamcast textures. pvrtex is part of KallistiOS (`utils/pvrtex`).<br>
The container is selected by the output extension:

| Extension       | Container                                      | Written by                                                       |
| :-------------- | :--------------------------------------------- | :--------------------------------------------------------------- |
| `.dtex`, `.tex` | DTEX. Both extensions contain identical bytes. | Texture Converter, sprite sheets, material export (`.dtex` only) |
| `.dt`           | DT                                             | Texture Converter, sprite sheets                                 |
| `.pvr`          | PVR (PVRT)                                     | Texture Converter, sprite sheets                                 |
| `.pal`          | DPAL palette                                   | Texture Converter, sprite sheets, beside paletted textures       |

After each export, the Studio decodes the new file with its own reader. It publishes the file only if the decode succeeds and the mip chain is complete.<br>
The upstream definitions are in pvrtex: `file_tex.c` (DTEX), `file_dctex.h` and `file_dctex.c` (DT), `file_pvr.c` (PVR), and `palette.c` (DPAL).

### 2.2 Pixel data

All containers share this pixel data. Only the header differs.

**Pixel format codes** (3 bits):

| Code | Format                 | Bits per texel |
| :--: | :--------------------- | :------------: |
|  0   | ARGB1555               |       16       |
|  1   | RGB565                 |       16       |
|  2   | ARGB4444               |       16       |
|  3   | YUV422                 |       16       |
|  4   | Normal map (spherical) |       16       |
|  5   | 4-bit palette index    |       4        |
|  6   | 8-bit palette index    |       8        |

**16-bit texels** are little-endian words. Bit 15 is the most significant bit.

| Format     | Bits 15 to 0                                                                                 |
| :--------- | :------------------------------------------------------------------------------------------- |
| ARGB1555   | A (15), R (14 to 10), G (9 to 5), B (4 to 0). A = 1 is opaque.                                        |
| RGB565     | R (15 to 11), G (10 to 5), B (4 to 0). Always opaque.                                                 |
| ARGB4444   | A (15 to 12), R (11 to 8), G (7 to 4), B (3 to 0).                                                       |
| YUV422     | Y in bits 15 to 8. Bits 7 to 0 hold U in the first texel of a horizontal pair and V in the second. |
| Normal map | Bits 15 to 8: elevation. Bits 7 to 0: azimuth = value ÷ 256 × 360°.                                |

- The Studio decodes the normal map with angle from the Z axis = (255 − bits 15 to 8) ÷ 255 × 90°.
- The Studio decodes YUV422 with U′ = U − 128, V′ = V − 128, then R = Y + 1.375 V′, G = Y − 0.34375 U′ − 0.6875 V′, B = Y + 1.71875 U′. In a mipmapped YUV422 texture, the Studio decodes the 1 × 1 level as RGB565.

**Paletted texels:** 8-bit data stores one index per byte. 4-bit data stores two indices per byte: the first texel in the low 4 bits and the next texel in the high 4 bits. The colors are in a separate [palette file](#29-palette-pal).

### 2.3 Texel order

**Linear:** row by row, starting at row 0.

**Twiddled:** for a square texture, the index of texel (x, y) interleaves the coordinate bits. Bit 0 of the index is bit 0 of y, bit 1 is bit 0 of x, bit 2 is bit 1 of y, and so on.<br>
For a rectangle, let `side` = the smaller of width and height. The texture is a row or column of `side × side` squares, stored one after another along the longer axis. Each square is twiddled.

```text
index = square × side² + interleave(y mod side, x mod side)
square = x ÷ side   (width ≥ height)
square = y ÷ side   (height > width)
```

Paletted textures are always twiddled.

### 2.4 Mipmap layout

A mipmapped texture is square with a power-of-two side.<br>
Levels are stored **smallest first**, after **three texels of padding**:

```text
[3 texels padding] [1×1] [2×2] [4×4] ... [N×N]
```

For a 16-bit format, the 1 × 1 level starts at byte 6. Each level of side `s` starts at byte `6 + 2 × (1 + 4 + 16 + … + (s/2)²)`.<br>
For 8-bit and 4-bit palette formats, scale the same texel offsets by 1 byte and 0.5 byte per texel. The 1 × 1 level of a 4-bit texture is in the high 4 bits of byte 1.<br>
Each level is twiddled.

Total uncompressed size in texels: `N² × 4/3 + 3`, rounded down. A non-mipmapped texture stores only the base level.

### 2.5 VQ compression

A VQ texture stores a **codebook** and then **indices**.

- A codebook entry is 8 bytes. For 16-bit formats, it holds four texels: a 2 × 2 block in twiddled order (x0 y0, x0 y1, x1 y0, x1 y1).
- Each index is one byte and selects one entry.
- To decode, replace each index with its 8-byte entry. The result is the uncompressed twiddled data from [2.3](#23-texel-order) and [2.4](#24-mipmap-layout). Number of indices = (uncompressed byte count + 7) ÷ 8.
- VQ textures are always twiddled.

### 2.6 DTEX and TEX

DTEX is the texture format of FDV Engine on Dreamcast. pvrtex calls it the `.TEX` format of the texconv tool.

**Header: 16 bytes.**

| Offset | Size | Field  | Value                                                                   |
| :----: | :--: | :----- | :---------------------------------------------------------------------- |
|   0    |  4   | Magic  | ASCII `DTEX`                                                            |
|   4    |  2   | Width  | Texture width rounded up to a power of two                              |
|   6    |  2   | Height | Texture height in texels                                                |
|   8    |  4   | Type   | Bit field, see below                                                    |
|   12   |  4   | Size   | Byte count of the data after the header, rounded up to a multiple of 32 |

**Type bits:**

| Bits  | Meaning                                                                                     |
| :---- | :------------------------------------------------------------------------------------------ |
| 31    | 1 = mipmapped                                                                               |
| 30    | 1 = VQ compressed                                                                           |
| 29 to 27 | Pixel format code ([2.2](#22-pixel-data))                                                   |
| 26    | 1 = linear (not twiddled). Ignored for palette formats.                                     |
| 25    | 1 = stride                                                                                  |
| 24 to 5  | 0                                                                                           |
| 4 to 0   | (true width ÷ 32) mod 32. Readers use it only when bit 25 is 1. The Studio reads 0 as 1024. |

**Data** starts at byte 16:

- VQ: a 2048-byte codebook (256 entries), then the indices.
- Not VQ: the texels.
- The data ends with zero padding up to `Size`. The file length is `16 + Size`.

### 2.7 DT

DT (`DcTx`) is the pvrtex native format. Its header is 32 bytes and its data is 32-byte aligned. pvrtex `file_dctex.h` defines it in full.

| Offset | Size | Field         | Value                                                       |
| :----: | :--: | :------------ | :---------------------------------------------------------- |
|   0    |  4   | Magic         | ASCII `DcTx`                                                |
|   4    |  4   | File size     | Total bytes including the header, a multiple of 32          |
|   8    |  1   | Version       | `0`. The Studio reads only version 0.                       |
|   9    |  1   | Header size   | (header bytes ÷ 32) − 1. Data starts at `(value + 1) × 32`. |
|   10   |  1   | Codebook size | Codebook entries − 1. VQ only.                              |
|   11   |  1   | Colors used  | Palette colors − 1. Palette formats only.                  |
|   12   |  2   | Width         | True width in texels                                        |
|   14   |  2   | Height        | True height in texels                                       |
|   16   |  4   | Type          | Bit field, see below                                        |
|   20   |  12  | Padding       | Zero                                                        |

**Type bits:**

| Bits  | Meaning                                       |
| :---- | :-------------------------------------------- |
| 31    | 1 = mipmapped                                 |
| 30    | 1 = VQ compressed                             |
| 29 to 27 | Pixel format code                             |
| 26    | 1 = linear. Not used for palette formats.     |
| 25    | 1 = stride (width not a power of two)         |
| 11    | 1 = partial: the height is not a power of two |
| 10 to 6  | Stride value: width ÷ 32                      |
| 5 to 3   | PVR width code: 0 = 8, 1 = 16 … 7 = 1024      |
| 2 to 0   | PVR height code, same scale                   |

DT supports a **small codebook**. The codebook holds `C` = (byte 10) + 1 entries.<br>
An index `i` selects entry `i − (256 − C)`.

### 2.8 PVR

PVR is the PowerVR texture format that PC viewers and many Dreamcast games use. pvrtex notes that its PVR support is incomplete and that it writes no `GBIX` chunk.

| Offset | Size | Field        | Value                                                             |
| :----: | :--: | :----------- | :---------------------------------------------------------------- |
|   0    |  4   | Magic        | ASCII `PVRT` (an optional `GBIX` chunk can precede it, see below) |
|   4    |  4   | Chunk size   | See below                                                         |
|   8    |  1   | Pixel format | Code from [2.2](#22-pixel-data)                                   |
|   9    |  1   | Layout       | See below                                                         |
|   10   |  2   | none         | 0                                                                 |
|   12   |  2   | Width        | Texels                                                            |
|   14   |  2   | Height       | Texels                                                            |
|   16   |  none| Data         | none                                                             | 

**Layout codes:**

|  Code   | Layout                                              |
| :-----: | :-------------------------------------------------- |
|  1 / 2  | Twiddled square, without / with mipmaps             |
|  3 / 4  | VQ, full 256-entry codebook, without / with mipmaps |
|  5 / 6  | 4-bit palette, without / with mipmaps               |
|  7 / 8  | 8-bit palette, without / with mipmaps               |
|    9    | Linear rectangle (no mipmaps)                       |
|   13    | Twiddled rectangle (no mipmaps)                     |
| 16 / 17 | VQ, small codebook, without / with mipmaps          |

The Studio also reads layouts 11 (linear) and 18 (twiddled with mipmaps, full padding). It does not write them.

**Rules:**

- A `GBIX` chunk is `GBIX`, a 4-byte length `L`, then `L` bytes. `PVRT` follows at offset `8 + L`.
- **Chunk size.** pvrtex writes the size of the whole chunk including its first 8 bytes. The Studio writes paletted PVR files itself, and for those it writes the size without the first 8 bytes, which is the PowerVR convention. A reader must accept both. If the value equals the bytes from `PVRT` to the end of the file, it includes the first 8 bytes.
- **Mipmaps without VQ.** pvrtex leaves out the first 4 bytes of the padding from [2.4](#24-mipmap-layout), so only 2 bytes of padding come before the 1 × 1 level. The Studio's paletted PVR files keep the full padding.
- **Small codebook size** (layouts 16 and 17) depends on the width `w`: `w` ≤ 16 → 16 entries. `w` ≤ 32 → 32 entries, or 64 with mipmaps. `w` ≤ 64 → 128 entries, or 256 with mipmaps. Larger → 256 entries.
- PVR VQ textures must be square. The Studio refuses stride and compressed palettes in PVR.

### 2.9 Palette (`.pal`)

The DPAL palette format stores the colors of a paletted texture.

| Offset |   Size    | Field   | Value                                                    |
| :----: | :-------: | :------ | :------------------------------------------------------- |
|   0    |     4     | Magic   | ASCII `DPAL`                                             |
|   4    |     4     | Count   | Number of colors, 1 to 256                              |
|   8    | 4 × Count | Colors | 32-bit ARGB8888: A (31 to 24), R (23 to 16), G (15 to 8), B (7 to 0) |

- The palette of `fire.dtex` is `fire.dtex.pal` (the full texture file name plus `.pal`). The Studio also finds `fire.pal` when the first name does not exist.
- A 4-bit texture uses at most 16 colors.
- Exporting to `.pal` writes only the palette.

### 2.10 Orientation

pvrtex can store rows bottom-up (`--flip-v`). The header has no flag for this.

- **Material bundles** write every DTEX file flipped. DTEX row 0 is the **bottom** of the picture. FDV maps row 0 to V = 0 and flips PNG files on load, so `<name>.png` and `<name>.dtex` produce the same result in FDV.
- **Texture Converter** and **sprite sheet** exports are not flipped unless you enable flipping.
- If your engine places V = 0 at the top of a PNG, reverse the rows of a bundle DTEX.

### 2.11 Studio export limits

| Rule                                                               | Applies to                              |
| :----------------------------------------------------------------- | :-------------------------------------- |
| Base size 8 to 1024 per side                                       | All Dreamcast containers                |
| Power-of-two sizes                                                 | Twiddled textures                       |
| Width 8, 16 or a multiple of 32, no palette, normal map or mipmaps | Stride textures                         |
| Square size                                                        | Mipmapped textures                      |
| RGB565, ARGB1555, ARGB4444 or AUTO, twiddled, no small codebook    | Material DTEX (surface and environment) |
| VQ compression when mipmaps are on                                 | Material DTEX (surface and environment) |
| Square size if environment mipmaps are on                          | Environment DTEX                        |

AUTO lets pvrtex select RGB565 for an opaque image, ARGB1555 for alpha that is only 0 or 1, and ARGB4444 otherwise.

---

## 3. Sprite sheet metadata (`.sprites.json`)

### 3.1 Files

**Export sprite sheet…** writes two files, and a third file for paletted textures:

| File                  | Contents                                                                     |
| :-------------------- | :--------------------------------------------------------------------------- |
| `<stem>.<ext>`        | The sheet: `.png`, `.jpg`, `.jpeg`, `.dtex`, `.tex`, `.dt`, `.pvr` or `.pal` |
| `<stem>.sprites.json` | The metadata                                                                 |
| `<stem>.<ext>.pal`    | The palette, for paletted Dreamcast formats only                             |

With **Include OBJ/MTL preview**, the export also writes `<stem>.obj` and `<stem>.mtl`, which show frame 1. A sheet that is not PNG or JPEG also gets a PNG copy, `<stem>.png`, for the MTL. See [OBJ/MTL preview](#5-objmtl-preview).

### 3.2 Layout

- Frames are in **row-major** order: left to right, then top to bottom.
- `columns` is the smaller of the requested columns and the frame count. `rows` = frameCount ÷ columns, rounded up.
- Cells have no padding or spacing.
- **Dreamcast containers** (not PNG, JPEG or PAL): the Studio enlarges the sheet to the next power of two, at least 8 and at most 1024 per side. With mipmaps, or with PVR and VQ, the sheet is square. The frames stay in the top-left area.
- Unused cells and any added area start as transparent black (all channels 0). Formats without alpha store them as black.
- Limits: 1 to 256 frames, 1 to 120 FPS, 1 to 256 columns. Image sheets: 8192 pixels per side and 16,777,216 pixels in total.

### 3.3 Keys

| Key                         | Type    | Meaning                                                                                                                                             |
| :-------------------------- | :------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| `format`                    | string  | Always `fdv-sprite-sheet`                                                                                                                           |
| `version`                   | integer | Always `1`                                                                                                                                          |
| `kind`                      | string  | `texture`, or `palette` when the export is a `.pal` file                                                                                            |
| `image`                     | string  | File name of the sheet, in the same folder. Absent when `kind` is `palette`.                                                                        |
| `palette`                   | string  | File name of the palette. Present for a `.pal` export or a paletted texture.                                                                        |
| `width`, `height`           | integer | Size of the stored sheet in pixels, including any power-of-two enlargement                                                                          |
| `frameWidth`, `frameHeight` | integer | Size of one frame in pixels                                                                                                                         |
| `frameCount`                | integer | Number of frames                                                                                                                                    |
| `fps`                       | integer | Playback rate in frames per second                                                                                                                  |
| `columns`, `rows`           | integer | Grid size in frames                                                                                                                                 |
| `order`                     | string  | Always `row-major`                                                                                                                                  |
| `origin`                    | string  | Always `top-left`. Frame coordinates count from the top-left texel of the sheet.                                                                    |
| `duration`                  | number  | Clip length in seconds: frameCount ÷ fps                                                                                                            |
| `framePixelsFlippedY`       | boolean | Present and `true` only when a Dreamcast sheet was exported with flipping. Each frame is flipped inside its own cell. Cell positions do not change. |
| `frames`                    | array   | One object for each frame, in playback order                                                                                                        |

Each `frames` entry:

| Key      | Type    | Meaning                                |
| :------- | :------ | :------------------------------------- |
| `x`, `y` | integer | Top-left corner of the frame in pixels |
| `w`, `h` | integer | Frame size in pixels                   |
| `time`   | number  | Start time in seconds: index ÷ fps     |

The loop does not repeat its first frame at the end.

### 3.4 Example

A four-frame PNG sheet with two columns, 64 × 64 frames at 8 FPS:

```json
{
  "columns": 2,
  "duration": 0.5,
  "format": "fdv-sprite-sheet",
  "fps": 8,
  "frameCount": 4,
  "frameHeight": 64,
  "frameWidth": 64,
  "frames": [
    { "h": 64, "time": 0.0, "w": 64, "x": 0, "y": 0 },
    { "h": 64, "time": 0.125, "w": 64, "x": 64, "y": 0 },
    { "h": 64, "time": 0.25, "w": 64, "x": 0, "y": 64 },
    { "h": 64, "time": 0.375, "w": 64, "x": 64, "y": 64 }
  ],
  "height": 128,
  "image": "flame.png",
  "kind": "texture",
  "order": "row-major",
  "origin": "top-left",
  "rows": 2,
  "version": 1,
  "width": 128
}
```

The Studio writes each frame object over several lines. The example shows one line per frame to save space.

---

## 4. Studio graph (`.fdvgraph`)

A `.fdvgraph` file is the editable source document of FDV Texture Studio. It is JSON.<br>
It stores the node graph, material settings, animation settings and the path of any imported image.

Do not use `.fdvgraph` files as a runtime format:

- Node types and parameters are internal Studio identifiers. This document does not define them.
- The key `fdvTextureGraph` holds the format version, `1` or `2`. The Studio refuses other versions. Graphs that use pinned controls or bypassed nodes are version `2` and need Texture Studio 0.2.0 or newer.
- A graph holds up to 256 nodes. Texture Studio 0.2.0 refuses graphs with more than 64 nodes.
- Its `material` object uses the `.fdvmat` keys and adds `writeDtex` and `envTexture` (export options for the environment map).
- Stability of the node data between Studio versions is Unspecified.

To use a material in another engine, read the `.fdvmat` and its textures.

---

## 5. OBJ/MTL preview

The Studio can also write a Wavefront OBJ file and its MTL material beside an FDV material or a sprite sheet. Tools such as FDV Editor, Blender and web viewers can open them for a quick preview. FDV Engine does not read them.

**Mesh.** One square, 1 m × 1 m, in the XZ plane at Y = 0, facing +Y. The four vertices are (−0.5, 0, 0.5), (0.5, 0, 0.5), (0.5, 0, −0.5) and (−0.5, 0, −0.5), in that order, with one normal (0, 1, 0).

**Texture coordinates.** The first vertex uses (left, bottom), then (right, bottom), (right, top) and (left, top).

| Export        | Left, bottom | Right, top                                                  |
| :------------ | :----------- | :---------------------------------------------------------- |
| FDV material  | 0, 0         | `tilingU`, `tilingV` (1 when the value is 0 or less)        |
| Sprite sheet  | 0, 1 − frame height ÷ sheet height | frame width ÷ sheet width, 1 (frame 1) |

**Material.** The MTL file contains one material with the name of the OBJ object:

| Key      | Value                                                        |
| :------- | :----------------------------------------------------------- |
| `Kd`     | The tint color (`color` red, green, blue). Sprite sheets: 1 1 1. |
| `Ks`     | `0 0 0`                                                      |
| `d`      | The tint alpha. Sprite sheets: 1.                            |
| `illum`  | `0` for Unlit materials and sprite sheets, otherwise `1`     |
| `map_Kd` | The PNG or JPEG texture beside the OBJ                       |

Blend modes, alpha cutoff, wrapping, scrolling, fog, depth test, two-sided drawing, mip bias, environment maps and Dreamcast textures have no MTL equivalent. They are only in the `.fdvmat`. MTL has no animation: play sprite frames with the `.sprites.json`.

**Names.** OBJ and MTL readers split names and file references at spaces, and `#` starts a comment. The Studio refuses an OBJ/MTL preview when the material name or the sprite sheet file name contains a space or `#`.

