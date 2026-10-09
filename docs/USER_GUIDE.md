# FDV Texture Studio · User guide

[← Back to the showcase](../README.md)

## Start here

Download a package from the repository's **Releases** page. On Windows, run the
installer or extract the portable ZIP and launch `fdv-texture-studio.exe`.
On Linux, install the DEB or extract the portable ZIP and run
`launch-texture-studio.sh`.

The release includes the native PVR encoder. You do not need a compiler, KOS,
or WSL to use the application. Linux needs glibc 2.38 or newer, X11/XWayland,
and an OpenGL 3.3 driver. If the status bar reports **CPU rendering fallback**,
the system is using software rendering; check your graphics driver.

## Create and edit

Choose a preset in **GPU Creator**, then select a resolution and seed.
The preset menu searches names, categories, and descriptions. Each preset is an
editable graph: select a node to adjust its settings and connections.

| Action                 | Control                                                                 |
| :--------------------- | :---------------------------------------------------------------------- |
| Pan the graph          | Drag with the middle or right mouse button.                             |
| Zoom the graph         | Mouse wheel.                                                            |
| Connect nodes          | Drag between pins, or click each pin in turn.                           |
| Add a connected node   | Drag a pin onto empty canvas, then search for a node.                   |
| Disconnect an input    | Right-click its pin.                                                    |
| Select several nodes   | Shift-click or drag a selection box on empty canvas.                    |
| Group a selection      | Press **C** or choose **Comment**.                                      |
| Inspect a node's image | Double-click the node.                                                  |
| Open node actions      | Right-click the node.                                                   |
| Enter a precise value  | Double-click or Ctrl+click a numeric slider.                            |
| Reset one setting      | Click its reset arrow, or right-click and choose **Revert to default**. |

Nodes have dark colors by category: green for math and masks, red for material
patterns, burnt orange for generators, purple for color and channels, slate blue
for filters, ochre for normals and lighting, teal for coordinates, plum for
animation, and indigo for textures.

Use **Texture sample** to combine independent images or preset snapshots.
**Source image (shared)** uses the graph's shared imported image. Save with
**Save graph** to retain your edits. Keep copied source images beside the graph
when moving it to another folder or computer.

Use **Presets → Custom → Save current as custom preset...** to add your graph
to the library. Selecting a custom preset creates an unsaved working copy.

## Preview materials

Switch between **Texture** and **3D material** above the preview. In 3D, choose
**Sphere**, **Cube**, or **Plane**, drag to orbit, and use the wheel to zoom.
**Reset view** restores the camera. Enable **2 × 2 tiling** to inspect repeats.
This preview setting does not change your exported texture.

Inspect Material output, Albedo, Height, and Normal when the graph provides them.
Texture export uses the selected preview; FDV material export uses Material output.

Many presets bake shading and highlights into the material image. Their FDV
material defaults to **Unlit** to preserve that appearance. Baked highlights stay
fixed in the texture; environment reflections use the FDV material settings.

## Animate and export a sprite sheet

Choose an animated preset, or connect **Time** to animation-capable nodes.
Use Play/Pause and the frame slider to inspect the clip. **Animation...** sets
frame count, FPS, sheet columns, and preview speed. Preview speed does not change
exported timing.

Choose **Export sprite sheet...** and select the format. PNG preserves transparency.
The export includes a `.sprites.json` sidecar describing each frame's rectangle,
FPS, and duration. Frames run left to right, then top to bottom. The repeated loop
endpoint is omitted, and unused PNG cells remain transparent.

Image atlases are limited to 8192 pixels per edge and 16 million pixels in total.
Direct Dreamcast atlases are padded to supported power-of-two dimensions, up to
1024 × 1024. Use the **Layout helper** for a 16-frame, four-column starting layout.
Atlas cells have no padding; sample within each frame's rectangle in your game.

## Convert and compare

Open an image or Dreamcast texture in **Texture Converter**, or select
**Use in converter** from the creator. Resize explicitly before generating a mip
chain. Choose a filter and edge handling; inspect individual mip levels or replace
a level with an image of the same size.

Set the encoding options, then use **Encode comparison** to inspect the actual
encoded and decoded pixels. Compare side by side, use a wipe, or amplify the RGB
difference. Supported encoding combinations depend on the selected container.
Paletted textures need their palette sidecar. RGB565 and YUV422 discard alpha.

The export dialog's **Format** selection controls the extension. Renaming an image
to `.pvr` does not convert it. PNG/JPEG export writes the selected mip level;
Dreamcast texture export can retain the authored mip chain.

## Export an FDV material

Use the **FDV material** panel for runtime settings such as surface identity,
alpha mode, wrapping, repeats, scrolling, tint, and environment reflections.
**Export FDV material** creates a named folder with a `.fdvmat` document,
textures, optional DTEX files, and the creator graph. Import that folder through
FDV Editor's material library workflow.

Keep the bundle together: its asset paths are relative. Environment images are
included and have their own size, filtering, and Dreamcast encoding settings.
GLB/glTF and OBJ/MTL export are not currently available.

## Report a problem

Include the app version, operating system, graphics renderer shown in the status
bar, steps to reproduce, and a screenshot where useful. Attach a small graph or
texture only if you are comfortable sharing it publicly.
