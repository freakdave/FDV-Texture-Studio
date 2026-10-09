# Release notes

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
