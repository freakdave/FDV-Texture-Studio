# FDV Texture Studio · User guide

[← Back to the README](../README.md)

## What do you want to do?

| Task | Go to |
| :--- | :--- |
| Make your first texture | [Create a texture](#create-and-edit) |
| Save or find a preset | [Preset library](#find-and-reuse-presets) |
| Connect, copy, or bypass nodes | [Edit a graph](#edit-a-graph) |
| Check a material or mask | [3D preview](#preview-materials) · [Texture channels](#inspect-texture-channels) |
| Export an animation | [Animation and sprite sheets](#animate-and-export-a-sprite-sheet) |
| Encode a Dreamcast texture | [Convert and compare](#convert-and-compare) |
| Export to FDV Editor | [FDV material bundles](#export-an-fdv-material) |
| Configure autosave or recover work | [Preferences and recovery](#preferences-autosave-and-recovery) |

[Installation](#start-here) · [Updating](#update-an-existing-installation) · [Shortcuts](#shortcuts)

## Start here

Download a package from [GitHub Releases](https://github.com/freakdave/FDV-Texture-Studio/releases).

| Package | How to start |
| :--- | :--- |
| **Windows installer** | Run Setup, then open FDV Texture Studio. |
| **Windows portable ZIP** | Extract the ZIP and run `fdv-texture-studio.exe`. |
| **Linux DEB** | Install the package, then open FDV Texture Studio. |
| **Linux portable ZIP** | Extract the ZIP and run `launch-texture-studio.sh`. |

You need a 64-bit desktop and an OpenGL 3.3 driver.<br>
Linux also requires glibc 2.38 or newer and X11 or XWayland.<br>
The native PVR encoder is included.

If the status bar reports **CPU rendering fallback**, check your graphics driver.

## Update an existing installation

Close Texture Studio before updating. There is no need to uninstall first.

- **Windows installer:** run the new Setup file. It replaces the application files and keeps your shortcut preference.
- **Linux DEB:** install the newer package with your package manager.
- **Portable ZIP:** extract the complete new ZIP into a fresh folder. Keep any graphs and imported images stored in the old folder.

Saved graphs and custom presets are kept when updating installed packages.

<details>
<summary>Linux install command and custom preset locations</summary>

From the download folder, run:

```sh
sudo apt install ./fdv-texture-studio_0.2.0_amd64.deb
```

Custom presets are stored outside the installation folder:

- **Windows:** `%LOCALAPPDATA%\FDV Texture Studio\presets`
- **Linux:** `$XDG_DATA_HOME/fdv-texture-studio/presets`, normally `~/.local/share/fdv-texture-studio/presets`

Files supplied with the application are replaced by the update.

</details>

## Create and edit

1. Open **GPU Creator** and choose a preset.<br>For a blank starting point, choose **New graph**. It creates a Checker node assigned as the material output.
2. Set the resolution and seed.
3. Select a node to edit its settings in the inspector.
4. Switch between **Texture** and **3D material** to check the result.
5. Choose **Export texture...** to save an image.<br>Use **Save graph** to retain the editable `.fdvgraph` document.

<img src="media/guide/creator.png" width="1080" alt="Wet mud in GPU Creator with preset controls at upper left and the selected Material pigment node's color ramp below">

_The inspector shows the selected node's settings. Here, Material pigment controls the colors._

<details>
<summary>Import images into a graph</summary>

Use **Texture sample** for independent images or preset snapshots.<br>
**Source image (shared)** uses the graph's shared imported image.

Keep copied source images beside the saved graph when moving it to another folder or computer.

</details>

## Find and reuse presets

1. Click **Browse presets...** above the preset menu.
2. Search by name or description, or filter by category.<br>Mark frequently used presets as favorites.
3. Hover a thumbnail to preview motion. Click it to load the preset.

<img src="media/guide/preset-browser.png" width="1080" alt="Preset browser showing material thumbnails, category filters, and favorites">

Custom presets also have thumbnails, generated from their saved graphs and imported images.

### Save or delete a custom preset

- **Save:** click **Save preset…** below the graph save buttons to add the current graph to the library.
- **Load:** choose the custom preset to create an unsaved working copy.
- **Delete:** click **Delete preset…**, choose a preset, and confirm **Delete**.<br>The current graph and source images are kept.

These actions are also available under **Presets → Custom**.

## Edit a graph

Drag between pins to connect nodes.<br>
Drag a pin onto empty canvas to open a menu of compatible nodes, then choose a result to connect it automatically.

<img src="media/acid-node-graph.gif" width="1000" alt="Acid bubbles graph with animated thumbnails showing each node's output">

_Double-click a node to inspect its output in the main preview._

### Follow and organize connections

<img src="media/guide/node-editing.png" width="1180" alt="Highlighted wire between Checker and Blend, gold output star, green reroute, and red missing-input ring">

| Indicator or action | Meaning |
| :--- | :--- |
| **Gold star** | This node is the material output. |
| **Red input ring** | A required input is missing. |
| **Hover a wire** | Highlight its path and show the connected node and input names. |
| **Right-click a wire** | Disconnect it, insert a Reroute, or find either endpoint. |
| **Drag a Reroute** | Reposition the wire without changing its values. |

To insert a node, drag its header until the node's center crosses a compatible wire.<br>
Release when the insertion hint appears. Hold **Alt** to move without inserting.

<details>
<summary>Connection rules and thumbnail controls</summary>

The connected-node menu names the input it will use.<br>
Enable **All nodes** to see other combinations, or drag backwards from an input to create its source.

Wire insertion requires an available compatible input.<br>
Existing inputs are kept. Connections that would introduce a cycle are rejected.<br>
Moving and inserting a node is one Undo step.

Right-click a node and choose **Freeze thumbnail** to stop its thumbnail updates.<br>
Choose **Resume thumbnail** to enable updates again.<br>
This does not affect the node's output or exports.

</details>

### Copy nodes between graphs

1. Select nodes and press **Ctrl+C**.
2. Open another graph or choose a preset.
3. Point at the canvas and press **Ctrl+V**.

Settings, images, comment membership, and connections within the selection are copied.<br>
Connections to unselected nodes are left open. The clipboard lasts while this application instance is open.

<details>
<summary>What changes when pasting or duplicating?</summary>

Pasted nodes use the destination graph's seed, animation settings, and resolution.<br>
The destination material output stays unchanged, so pasting may not reproduce the source material's final appearance.<br>
A copied shared source image becomes an independent Texture sample.

**Ctrl+D** duplicates nodes within the current graph and retains their incoming connections.<br>
Undo removes an entire paste or duplication in one step.

</details>

### Bypass a node

Right-click a compatible node and choose **Bypass node**, or use its inspector checkbox.<br>
It passes through its first input while keeping its settings and connections.

<img src="media/guide/bypassed-node.png" width="1080" alt="Bypassed Multiply node with black and yellow stripes and a BYPASSED label">

Bypass changes the final texture and exports. Toggle it off or use Undo to restore the operation.<br>
Generators and operations without a suitable source input do not offer bypass.

### Pin frequently used parameters

Pinned controls let you adjust selected node settings from one place.

1. Select a node and click **Pin parameter...** in its inspector.
2. Choose a parameter. It appears under **Pinned controls**, below the FDV material settings.
3. Use **...** beside the control to rename it, reorder it, find its node, or unpin it.

<img src="media/guide/graph-controls.png" width="1080" alt="Pinned Surface scale and Fine detail controls above the node inspector">

A pinned control edits the original parameter. It does not create a node or a separate value.<br>
Unpinning removes only the shortcut. Pinned controls are saved with the graph.

Use **Search all properties...** to find parameters by name or node.<br>
Edit matching values in the results, or use **Find node** to locate their owner.

### Save reusable node groups

1. Select nodes, right-click, and choose **Save selection as group...**.
2. Enter a name and click **Save selected nodes**.
3. Open **Settings > Node groups...** to insert or delete library groups.

<img src="media/guide/node-groups.png" width="1080" alt="Node group library with a saved filter and Insert and Delete buttons">

Connect the named **Input:** and **Output:** points after inserting a group.<br>
Each copy contains editable nodes, imported images, and pinned controls.<br>
Changing a library group does not change copies already inserted into graphs.

<details>
<summary>Node category colors</summary>

<img src="media/guide/node-categories.png" width="1080" alt="Nine fully colored nodes showing olive generators, green math, purple color, blue filters, amber lighting, red coordinates, rose animation, cyan textures, and orange patterns">

| Category | Color |
| :--- | :--- |
| Generators | Olive |
| Math and masks | Green |
| Color and channels | Purple |
| Filters | Blue |
| Normals and lighting | Amber |
| Coordinates | Red |
| Animation | Rose |
| Textures | Cyan-blue |
| Material patterns | Orange |

</details>

### Shortcuts

Graph shortcuts apply while the canvas has focus. Text fields use normal text-editing shortcuts.

| Action | Control |
| :--- | :--- |
| New graph | **Ctrl+N** |
| Copy / paste nodes | **Ctrl+C** / **Ctrl+V** |
| Duplicate nodes | **Ctrl+D** |
| Select all nodes | **Ctrl+A** |
| Select several nodes | **Shift-click**, or drag a selection box on empty canvas |
| Undo / redo | **Ctrl+Z** / **Ctrl+Y** |
| Add a comment around the selection | **C** or **Comment** |
| Frame the selection / find the output | **F** / **Home** |
| Pan / zoom | Middle or right mouse drag / mouse wheel |
| Inspect a node's output | Double-click the node |
| Bypass a compatible node | Right-click the node → **Bypass node** |
| Disconnect an input | Right-click its pin |
| Enter an exact value | Double-click or **Ctrl+click** a numeric slider |
| Reset one setting | Reset arrow, or right-click → **Revert to default** |

## Preview materials

1. Select **3D material** above the preview.
2. Choose **Sphere**, **Cube**, or **Plane**.
3. Drag to orbit and scroll to zoom. **Reset view** restores the camera.
4. Enable **2 × 2 tiling** to check repeats, or **Material motion** to preview scrolling.

<img src="media/guide/material-preview.png" width="1080" alt="Wet mud on a sphere with mesh selection, tiling, and camera controls above the preview">

Texture export uses the selected preview output.<br>
FDV material export uses **Material output**. Tiling the preview does not change exports.

### Keep editing responsive

Choose **Auto** preview quality to reduce working resolution during editing and playback.<br>
The default limit is 512 × 512. Change it in **Settings > Preferences...**.

Idle previews return to the graph resolution. **Full quality** always uses that resolution.<br>
Exports use the requested document resolution. The toolbar shows preview and export dimensions.

<details>
<summary>Wet materials, alpha masks, and reflections</summary>

Wet presets can use alpha as a reflection mask.<br>
**Reflect holes (add)** and **Mirror holes (replace)** use inverse texture alpha:

- Opaque pixels receive no environment reflection.
- Transparent pixels receive full reflection.
- Intermediate alpha gives partial reflection.

Select **Dryness / reflection mask** in wet presets to adjust reflective areas.<br>
The flat texture view may show a checkerboard even when the 3D material is opaque.

Many presets include baked shading and highlights and default to **Unlit**.<br>
Baked highlights stay fixed in the texture. Environment reflections use the FDV material settings.

</details>

### Check runtime settings in the preview

Expand **Preview scene** above the 3D view to inspect fog, depth testing, and mip filtering.

<img src="media/guide/runtime-preview.png" width="1080" alt="Brick material with preview fog, an occluding panel, and mipmapped sampling">

| Control | What to check |
| :--- | :--- |
| **Preview fog** | Adjust color and near/far distances. Enable the material's **Fog** switch. Unlit materials ignore fog. |
| **Show occluder** | Toggle the material's **Depth test**. With it off, the material draws through the panel. |
| **Mipmapped preview** | Inspect distance filtering and the surface and environment mip biases. |

These scene controls are for inspection and are not exported as material settings.

<details>
<summary>Mip bias and encoded texture quality</summary>

Surface bias 4 is neutral. Lower values are sharper and higher values are softer.<br>
Environment bias 0 inherits the surface bias.

Mip sampling follows FDV Editor's desktop convention.<br>
The preview generates its own mip levels and does not show DTEX quantization or VQ compression.

Use **Encode comparison** to inspect encoded pixels, then check final rendering on Dreamcast.<br>
The export's **Mipmaps** option controls whether the DTEX file includes mip levels.

</details>

## Inspect texture channels

1. Switch to **Texture** view and choose an output such as Albedo, Height, or Normal.
2. Use **Inspect** to select RGBA, RGB, Red, Green, Blue, or Alpha.
3. Hold **Ctrl** over the image to read a pixel's RGBA values and coordinates.

<img src="media/guide/channel-inspector.png" width="1080" alt="Wet mud alpha channel displayed as a grayscale reflection mask">

Inspection does not change the graph or exports.<br>
The pixel readout uses the displayed working resolution. Choose **Full quality** to inspect at export resolution.

## Animate and export a sprite sheet

1. Choose an animated preset, or connect **Time** to animation-capable nodes.
2. Open **Animation...** and set frame count, FPS, and sheet columns.<br>The **Layout helper** provides a compatible starting layout.
3. Use Play/Pause and the frame slider to inspect the clip.
4. Choose **Export sprite sheet...**, select a destination and format, then save.

<img src="media/guide/animation.png" width="480" alt="Animation dialog with frame count, FPS, columns, layout helper, duration, and atlas dimensions">

_Preview speed changes playback only. Exported timing uses the clip FPS._

PNG preserves transparency.<br>
Keep the exported image and its `.sprites.json` sidecar together. The JSON describes frame rectangles, FPS, and duration.

<details>
<summary>Export dialog and atlas limits</summary>

<img src="media/guide/sprite-export.png" width="800" alt="Sprite sheet export dialog with filename, PNG format, and Save button">

Frames run left to right, then top to bottom.<br>
The repeated loop endpoint is omitted. Unused PNG cells remain transparent.

| Atlas type | Limit |
| :--- | :--- |
| Image | 8192 pixels per edge and 16 million pixels in total |
| Direct Dreamcast export | Supported power-of-two dimensions, up to 1024 × 1024 |

The Layout helper offers a 16-frame, four-column starting layout.<br>
Atlas cells have no padding. Sample within each frame's rectangle in your game.

</details>

## Convert and compare

1. Open a file in **Texture Converter**, or choose **Use in converter** from GPU Creator.
2. Set the size before generating mip levels. Choose a filter and edge handling.
3. Select the container and encoding options.
4. Click **Encode comparison** to check the encoded result.<br>Use **Compare** for side-by-side, wipe, or amplified RGB difference views.
5. Export with the required **Format** in the save dialog.

<img src="media/guide/converter.png" width="1080" alt="Texture Converter with mipmap and encoding controls beside the original and decoded texture comparison">

**Alpha matters:** RGB565 and YUV422 discard alpha.<br>
Use ARGB4444 for smooth transparency or reflection masks, or ARGB1555 for binary cutouts.

<details>
<summary>Supported formats and mip levels</summary>

| Format | Use |
| :--- | :--- |
| PNG / JPEG | Image import and export. PNG supports alpha. |
| DTEX / TEX, DT, PVR | Dreamcast texture containers |
| PAL | Palette sidecar for paletted textures |

Encoding options include RGB565, ARGB1555, ARGB4444, YUV422, PAL4, PAL8, BUMP, and NORMAL.<br>
Available combinations depend on the selected container. Keep palette sidecars with their textures.

Select a mip level to inspect it, or replace it with an image of the same size.<br>
PNG/JPEG export writes the selected mip level. Dreamcast texture export can retain the authored mip chain.

The save dialog's **Format** controls the extension.<br>
Renaming an image to `.pvr` does not convert it.

</details>

## Export an FDV material

1. Open **FDV material** below **Use in converter**.
2. Name the material and set alpha, wrapping, scrolling, tint, and reflections as needed.
3. Review any validation messages.<br>For example, **Use ARGB4444** corrects an encoding that would discard a reflection mask.
4. Click **Export FDV material...** and choose a parent folder.

<img src="media/guide/fdv-material.png" width="580" alt="FDV material settings with material name and Export FDV material button">

The export creates a folder containing the `.fdvmat`, textures, and creator graph when available.<br>
Keep this folder together and import it through FDV Editor's material library.

### Include Dreamcast DTEX

DTEX is the texture format used by FDV Engine on Dreamcast.<br>
This checkbox applies to **Export FDV material…**.

| Setting | Exported bundle |
| :--- | :--- |
| **Off** | References PNG textures for desktop and image-based workflows. |
| **On** | References a surface DTEX and an active environment-map DTEX. PNG copies and the creator graph, when available, are kept. |

Choose the surface pixel format, mipmaps, and VQ compression below the checkbox.<br>
Environment maps have separate controls under **Environment texture options**.<br>
Use an alpha-capable format for transparency or reflection masks. RGB565 cannot preserve them.

<details>
<summary>Bundle contents and validation</summary>

Bundle asset paths are relative. Environment images are included with their own size, filtering, and encoding settings.<br>
GLB/glTF and OBJ/MTL export are not available.

Validation checks cover alpha loss, missing environment images, reflective-hole settings, DTEX sizes, and unsupported layouts.<br>
Apply suggested corrections beside each message as needed. Export validates the request before opening the destination picker.

</details>

## Preferences, autosave, and recovery

Open **Settings > Preferences...**.<br>
Choose **Apply** to save settings for future sessions, or **Cancel** to discard changes.

<img src="media/guide/preferences.png" width="1080" alt="Preferences dialog with autosave interval, recovery count, working resolution, thumbnail rate, grid, and Undo history">

| Setting | Purpose |
| :--- | :--- |
| **Enable autosave** | Save separate recovery copies, including untitled graphs. |
| **Interval** | 1 to 60 minutes between autosaves. Default: 2 minutes. |
| **Recovery versions per graph** | Keep 1 to 50 versions. Default: 10. |
| **Working resolution** | Limit editing and animation previews to 256, 512, or 1024 pixels. |
| **Thumbnail updates per second** | Set animated thumbnail updates from 1 to 30 per second. |
| **Pause thumbnails during interaction** | Prioritize dragging, orbiting, and active edits. |
| **Node editor defaults** | Set thumbnail and grid visibility and keep 10 to 200 Undo steps. |

### Restore an autosave

1. Open **Settings > Recover graph...**, or use the recovery prompt after an interrupted session.
2. Choose a version and click **Restore copy**.
3. Save the recovered graph under the name and location you want.

Autosave includes imported, sampled, and environment images.<br>
It does not overwrite your saved graph. Changes since the last autosave cannot be recovered.

<details>
<summary>Recovery retention and graph compatibility</summary>

Older versions are removed after a successful autosave when the retention limit is reached.<br>
**Dismiss startup reminders** keeps the copies but stops prompting for those sessions.<br>
Preferences shows the recovery folder path, which you can copy to your file manager.

Graphs using pinned controls, bypass, or reroutes require Texture Studio 0.2.0 or newer.

</details>

## Report a problem

[Open an issue](https://github.com/freakdave/FDV-Texture-Studio/issues) with:

- App version and operating system
- Graphics renderer shown in the status bar
- Steps to reproduce the problem
- A screenshot or small example graph, if useful and suitable for public sharing
