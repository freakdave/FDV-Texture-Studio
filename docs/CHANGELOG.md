# Release notes

## 0.4.0

### Command line

- Export maps without a window with `fdv-texture-studio --export <graph | folder | preset:Name> --out <folder>`. Choose `--size`, `--maps` (or `all`), `--format png|dtex`, `--dtex` (PNG and DTEX together) and `--recursive` for subfolders. It runs on Windows and Linux.
- Write Dreamcast DTEX from the command line with `--pixel`, `--vq`, `--mipmaps`, `--kaiser` and `--pvrtex`. The files match the Export panel and stop at 512 pixels.
- A folder exports every graph with one OpenGL context. A failing graph does not stop the rest, and the run ends with a summary.

### PBR maps

- A graph can supply roughness, metallic, ambient occlusion (`ao`), emissive and opacity maps, next to albedo, height and normal. Assign them with the second row of **Use this node as** buttons. The Preview has a button for each. The `.fdvgraph` file stores them in optional keys, so older files still load. The `.fdvmat` has no slot for them.
- Presets assign roughness and ao where they already compute them.

## 0.3.0

### Material preview

- Rename the **3D material** preview tab to **FDV material**. The **Texture** tab shows what **Export texture** and **Export sprite sheet** save. The **FDV material** tab shows what **Export FDV material** saves.
- Switch a material between lit and **Unlit** with the gold-framed sun in the FDV material view.
- Toggle **Mipmapped preview**, **Preview fog**, **Show occluder**, and **Environment map** from icons in the view. A legend marks which icons change the FDV material and which are preview only.
- Hide or show the material's environment map from the view to compare. The material keeps its setting.
- Use the FDV icon as the default environment reflection.
- Preview **Tiling** at 1 × 1, 2 × 2, or 4 × 4 in both Texture view and the 3D material preview.
- Remove the dashed lines on cube edges and the sphere's seam in **Mipmapped preview**. Each pixel now selects its mip level from its own surface, as a rendered triangle does.
- Edit the FDV material settings in their own **FDV material** panel, by default a full-height column on the right. They were in a collapsed section below the graph controls. Narrow color rows show just the swatch, which still opens the picker.

### Workspace

- Use Roboto, the Unreal Engine editor font, for the interface instead of the built-in pixel font.
- Rename **GPU Creator** to **Texture Creator**, and **Use in converter** to **Use in Texture Converter**.
- Move **Settings** and **Help** to the right end of the menu bar, beside the new **View** menu.
- Dock, float, tab, and resize the **Controls**, **Node inspector**, **Preview**, and **Node graph** panels. Close a panel with its **×** and show it again from **View**. The layout is saved in the user folder. **View > Reset layout** restores the default.
- Fill the window with the node graph with the **Focus graph** button in the node graph toolbar, **View > Focus graph**, or **F11**. The same button reads **Exit focus** to return. As in Unreal Engine's material editor, a 3D material preview sits in the graph's corner. Drag it to orbit, click its shape icon to switch between sphere, cube, and plane, or hide it. The corner preview also appears whenever the Preview panel is hidden. The node inspector sits in a resizable column beside the focused graph. Clear **Inspector** to hide it.
- The Texture Converter has its own panel layout.
- Save up to four named layouts with **View > Layouts > Save layout as...**, load them from the same menu, and remove them with **Delete layout**. A layout keeps the panel arrangement, which panels are shown, and Focus graph with its inspector column. Saving under an existing name replaces that layout. Layouts are stored in the `layouts` folder of the user folder.
- Start with the **New graph** Checker document, not the Brick preset.
- Show the **Recover graph** startup prompt once for each session that ended without a normal close, for example after a crash. Before, it reappeared on every launch until **Dismiss startup reminders** was clicked. That button is gone. The autosaved copies stay available in **Recover graph...**.
- Show **Export sprite sheet** and the Play, Frame, and Animation controls only when a Time node is connected to an output. This also works for custom graphs: an unconnected Time node does not count.

### Node editing

- Edit node colors with one color picker instead of separate red, green, blue, and alpha sliders: **Color**, **Color ramp** (Low color, High color), and **Foliage colors**. Click the swatch for a hue wheel, an alpha bar, and RGB, HSV, and hex fields. Pins, property search, and FDV material colors use the same picker.
- Show the node type in each node's header, so it reads the same in every graph. The graph's own name for the node moves to the footer in brighter text, with **OUTPUT** or **BYPASSED** at its right end.
- Pin a whole color. Pins of a single color channel from older graphs show the whole color.
- The node inspector shows **Use this node as** with one row of buttons: Output, Albedo, Height, and Normal. Each assigns which node supplies that map of the graph: Export saves the Output, and the Preview's Material output, Albedo, Height, and Normal buttons and Texture Sample nodes read them. The maps this node supplies are green. Each tooltip names the node that supplies the map now. Right-click reverts a map. **Preview node** and **Delete node** sit below.
- Build graphs of up to 256 nodes, four times the old limit. Node settings now reach the GPU through a uniform buffer, which OpenGL 3.3 guarantees on every GPU. Built-in presets keep 8 nodes free for your own additions.
- Graphs where a costly height feeds **Relief lighting**, **Normal**, or a filter no longer fail with "GPU evaluation budget". Those inputs render to their own texture first, so each sample now counts as one texture read.
- Record one Undo step per drag. A drag on a slider or color previously added one step for each frame.

### Performance

- Keep compiled GPU shaders between sessions in the `shader-cache` folder of the user folder (at most 256 MB, the oldest unused shaders are removed first). A graph's shaders change only when its nodes or connections change, so reopening a preset or a saved graph skips compiling. A driver update recompiles automatically. **Settings > Preferences > Shader cache** shows its size and clears it.
- Open the Toon presets in under a second. On an AMD integrated GPU, Toon cobblestone took 94 s to compile its node thumbnails and 2.2 s for its preview. Now it takes 0.6 s and 55 ms. Two causes are fixed below.
- Compile only the shader functions a graph uses. Every preview pass and node thumbnail carried the whole 52 KB function library before. Most now carry 5 to 17 KB.
- Render particle surfaces (Aggregate and Foliage) as their own pass for node previews and for previews up to 2048 × 2048. A node that read several of their channels inlined the particle loops once per channel, and one Toon cobblestone thumbnail took 19 s to compile.
- Show the shader state at the right end of the status bar, where text of changing length beside it cannot move it: red **Compiling shaders** while the preview or the node thumbnails compile GPU shaders, green **Shaders compiled** when all are ready. A preview compile blocks the window for a moment on large graphs, so the status appears one frame before it, and the preview keeps its last image meanwhile.
- Keep high resolutions such as 4096 × 4096 responsive. Clicks, wheel zoom, orbiting, panning, node moves, and FDV material changes no longer re-render the unchanged texture. Before, each of them caused a reduced render followed by a full-resolution render (about 190 ms at 4096 on an integrated GPU).
- Update animated node thumbnails at the rate set in **Preferences**, all in step. Before, they updated one node after another at about 4 to 13 frames per second whatever the setting. Thumbnails never update faster than the graph's Clip FPS.
- Use the reduced preview only while a held control, such as a slider drag, changes the texture, and during playback. Buttons, Undo, and loading render full resolution at once.

### Texture Converter

- **Use in Texture Converter** on an animated graph says which animation frame the converter holds as a still image. Creator playback continues after returning.
- Keep a texture's description in the converter when its mipmaps are rebuilt.

### Presets

- Raise the maximums for deep relief: **Relief depth** goes up to 1, **Occlusion radius** to 0.5, and **Crevice shading** to 2. **Height occlusion** depth goes to 1 and its strength to 4. **Normal** strength goes to 32, **Normal strength** to 16, **Baked specular** strength to 8, and **Diffuse**, **Specular**, and **Exposure** to 4. Saved graphs load unchanged.
- Add 8 **Sci-fi** presets: Sci-fi panels, Hex armor, Metal compartments, Ribbed siding, Vent grille, Glowing tech panel, Grimy space panels, and Hazard plating. Each has crisp machined panels split into irregular sub-panels at stepped heights, with screws, vents, louvers, pipe trenches, and hazard and label decals. Deep relief and long cast shadows give them depth. Each is weathered: fine grain, micro-scratches, worn edges, and dirt in the cavities. Bare metal carries baked highlights. In the FDV material, the environment reflects only through irregular alpha holes (Alpha holes: Reflect): chipped edges, scratches, bolts, and rubbed-clean blotches. It never covers whole plates. LED strips and indicators glow with a slight bloom.
- Add 8 **Stylized** toon presets: Toon brick, Toon cobblestone, Toon stone wall, Toon planks, Toon grass, Toon sand, Toon dirt, and Toon tiles. They are painted in the hand-painted game style: light comes from above with cool, saturated shadows and warm highlights, raised edges get a warm rim, and cavities turn deep and colored.
- Add the **Tyre marks** preset under **Terrain**. Two passes of rally tyres press ruts into a sampled preset, Snow by default. Choose the same preset in both base sample nodes to put the marks on Gravel, Mud, Sand or any other static preset. Each tyre stamps staggered tread blocks into the rut floor and pushes up soft lips. The base surface's own grain breaks the stamp and roughens the rut edges, so gravel looks gritty and snow stays crisp. Pinned controls set wheel spacing, tyre width, wander, track direction, rut and tread depth, grit and mark visibility. The marks tile seamlessly.
- **Pulsing glow** defaults to **Blended** surface alpha, so its soft halo fades smoothly. It used Cutout, which cut the halo into hard holes.

### Export

- Optionally write an OBJ/MTL preview quad with FDV materials and sprite sheets, for FDV Editor, Blender, and web viewers.
- Refuse FDV material DTEX that is uncompressed and mipmapped. FDV Engine on Dreamcast cannot load it. Correction buttons enable VQ compression or disable mipmaps.

### Documentation

- Add a file format reference for FDV materials, Dreamcast textures, and sprite sheet metadata, so other engines can read Studio exports.
- Refresh every user guide screenshot and the animated node graph for the 0.3.0 interface, and add screenshots of Focus graph, the color picker, and **Use this node as**.
- Add user guide sections for the workspace and saved layouts, colors, graph maps, and the shader status. Add the new Sci-fi and Stylized presets to the README.

## 0.2.0

### Node editing

- Copy and paste selections between graphs with **Ctrl+C** and **Ctrl+V**.
- Duplicate selections with **Ctrl+D**, preserving incoming connections.
- Start a Checker graph with **New graph** or **Ctrl+N**.
- Identify node categories by their full background colors and the material output by its gold star.
- Highlight connections on hover and show their endpoint names.
- Right-click a connection to disconnect it, insert a Reroute, or find either endpoint.
- Release a connection on empty canvas to get node suggestions for the starting pin.
- Insert compatible nodes by dropping them onto wires. Hold **Alt** to move without connecting.
- Frame the selection with **F** and find the output with **Home**.
- Mark missing required inputs with red rings.
- Bypass compatible nodes without losing settings or connections. Bypassed nodes have black and yellow stripes.
- Pin node parameters as sidebar shortcuts and search properties across the graph.
- Save reusable node groups with named inputs, outputs, imported images, and pinned controls.

### Presets and workspace

- Browse presets with thumbnails, category filters, search, favorites, and hover animation.
- Generate custom preset thumbnails from saved graphs, including imported images.
- Save and delete custom presets from a dedicated row below the graph save buttons.
- Place FDV material settings directly below **Use in converter**, followed by pinned controls and the node inspector.
- Add **Settings > Preferences** for autosave intervals, recovery retention, Undo history, preview resolution, and thumbnail settings.
- Recover saved or untitled graphs from separate autosave copies that include their image dependencies.
- Add **Help > About** with the app version, author David Reichelt, and project link.

### Preview and performance

- Use **Auto** preview quality to reduce working resolution during editing and playback. Exports retain the requested resolution.
- Reuse unchanged branch thumbnails, pause updates during interaction, and limit sampled preset resolution within thumbnails.
- Freeze and resume individual node thumbnails.
- Show preview resolution, export resolution, and thumbnail update status together.
- Inspect RGB, alpha, or individual channels. Hold **Ctrl** over the image to read pixel values.
- Preview mip bias, scene fog, and depth testing against an occluder in the 3D view.
- Cache preview mipmaps while orbiting without changing source textures.
- Correct **Reflect holes** and **Mirror holes** masks so opaque areas receive no environment reflection.

### Material export

- Show incompatible export settings with correction buttons beside the relevant messages.
- Explain the PNG and Dreamcast DTEX bundle options beside **Include Dreamcast DTEX**.

### Documentation

- Refresh UI screenshots and the animated node graph for the current interface.
- Shorten the README feature overview and add a platform download table.
- Organize the user guide around tasks, numbered steps, and a shortcut table.
- Put advanced settings and format limits in expandable sections.

### Updating and compatibility

Close Texture Studio before updating. Installed packages do not need to be uninstalled first.<br>
For portable packages, extract the complete ZIP into a fresh folder and keep any graphs or images stored in the old folder.

Existing graphs remain readable in 0.2.0.<br>
Graphs using pinned controls, bypass, or Reroute nodes require 0.2.0 or newer.<br>
The node clipboard is local to the running application instance.
