# NoteDraw1

NoteDraw1 is a plugin for editing rendered note text and drawing directly on notes.

It is built as a surface layer: the same drawing and text-edit logic works on Obsidian reading view, source view, embedded note previews, and supported webview surfaces.

## Features

- Magic-wand header button for entering NoteDraw1 mode.
- In-place text editing in reading view.
- Source/edit view overlay using the same command entry.
- The original pen remains the default, with a separate speed-sensitive fountain pen that varies stroke width while keeping opacity stable.
- Reading view adds a note pen that uses fountain-pen dynamics while reserving clean Markdown flow space beneath the stroke.
- The original watercolor brush remains available, with text-aligned and auto-straight variants.
- Stroke selection, multi-select, movement, resize handles, and delete.
- Floating text, magnetic rectangles and circles, and three-point curved arrows.
- Text style toggles for bold, italic, underline, and boxed text.
- Text, rectangles, circles, and arrows are grouped under one Text and links section.
- Text and shape tools can select existing drawing elements before creating new ones.
- Text elements can be double-clicked to edit again, and text-panel style buttons apply to active rendered Markdown text when possible.
- Circular toolbar buttons sized for quick touch or mouse use.
- Palette changes recolor selected elements directly; without a selection, select mode keeps the palette closed.
- Selected elements can be copied and pasted across notes with styles, assets, layering, and bound connectors intact.
- The active tool, brush variant, text preset, zoom, color, width, opacity, and mind-map link choice persist across notes and restarts.
- Active pen and watercolor buttons use their current brush color as the button background.
- The palette has common color swatches plus an advanced color picker entry, and applies color changes to selected elements.
- Toolbar positioning stays below the Obsidian view header while scrolling.
- Lazy drawing-data loading to reduce note-open lag.
- Viewport-windowed canvas rendering with mobile pixel budgets for stable long-note performance.
- Inactive canvases stay out of the compositor, and stale view controllers are released on mode or file changes.
- Hidden Obsidian embed copies are not mounted, and offscreen embeds pause scroll-driven layout work until they approach the viewport.
- Runaway historical element coordinates are bounded and repaired instead of stretching a note to an unusable height.
- Responsive coordinates follow the Markdown content lane and nearby source lines across reading, source, desktop, and mobile layouts.
- Reading view uses non-reflowing visual zoom: original line breaks stay fixed, enlarged content can overflow horizontally, and two-axis panning keeps every NoteDraw1 layer aligned.
- Source/edit view keeps layout zoom, so text can reflow to the edited working width.
- Click-to-caret behavior inside active text blocks.
- Configurable drawing storage, with the plugin config folder remaining the default.
- Portable single-file sharing that keeps normal Markdown readable while embedding NoteDraw1 data and linked resources in a hidden block.
- Public API for scripts, other plugins, and AI agents.
- Drawings made inside embedded note previews are stored against the embedded note path, so opening that note shows the same layer.
- Embedded `![[Markdown notes]]` can be edited in place: direct editing in source view and the floating format toolbar in reading view.
- Webview surfaces get independent drawing files, so annotations do not bleed between pages.
- File-backed Canvas, PDF, image, database, and other workspace views retain their magic wand and drawing layer across internal rerenders.
- Imported images, videos, files, Markdown, and HTML can be placed as floating NoteDraw1 elements.
- Markdown files can be converted into rendered, editable NoteDraw1 mind maps with visible embeds, magnetically bound curved connectors, optional source-note updates, and source-note navigation.

## Storage

By default, new drawing files are stored here:

```text
<vault>/.obsidian/plugins/notedraw1/drawings/
```

The **NoteDraw1 data location** setting provides four choices:

- Plugin config folder (default): `<vault>/.obsidian/plugins/notedraw1/drawings/`
- Current note folder / `notedraw1`: `<note-folder>/notedraw1/<note>.notedraw1.json`
- Current note folder: `<note-folder>/<note>.notedraw1.json`
- Current Markdown file: a hidden, compressed NoteDraw1 block appended to the note

Changing the setting preserves old data and copies the active note's current drawing into the selected location. NoteDraw1 can still read existing config-folder, embedded, and legacy data and uses the newest valid copy.

**Share NoteDraw1 file** is available in the note menu and command palette. It creates a uniquely named `<note>.notedraw1.md` copy containing the readable Markdown body plus a hidden portable bundle with the drawing layer, NoteDraw1 attachments, internal linked files, and reachable HTTP/HTTPS resources. NoteDraw1 opens that copy in Obsidian reading view and waits for its Markdown, resources, and drawing layer to render before opening system sharing. The source note and existing copies are never overwritten. Normal Markdown readers ignore the hidden block; NoteDraw1 restores it when the file is opened in a compatible Obsidian environment.

## Migration

This independent fork uses the plugin id:

```text
notedraw1
```

If an older local prototype folder exists, NoteDraw1 can read its previous drawing JSON files and copy them into the new `notedraw1/drawings` folder on first access. The old files are not deleted.

## Manual Install

Copy these files into:

```text
<vault>/.obsidian/plugins/notedraw1/
```

Required files:

```text
main.js
manifest.json
styles.css
```

Then enable:

```text
Settings -> Community plugins -> Installed plugins -> NoteDraw1
```

## Source Build

NoteDraw1 now keeps source code under `src/` and builds the Obsidian runtime file at the repository root.

```bash
npm install
npm run build
```

Build output:

```text
main.js
```

The release package still uses the standard Obsidian plugin layout:

```text
main.js
manifest.json
styles.css
```

The source tree keeps `extras/` for support-code images used at build time. Release builds embed those images into `main.js`, so the installed plugin does not require separate image files.

## Settings

The settings page currently includes:

- Default pen color, width, opacity.
- Default watercolor color, width, opacity.
- UI language.
- NoteDraw1 data location.
- Toolbar top offset.
- Long-press delay, tap tolerance, selection hit padding, and selected-element opacity.
- Stroke smoothing, input sampling, save compaction, and auto-save delay.
- Reset buttons for brush defaults and layout/interaction defaults.
- Debug log toggle for troubleshooting text targeting.
- Two fixed support QR codes shown from bundled assets with embedded fallback.

## Extension API

NoteDraw1 exposes a small API from the plugin instance:

```js
const api = app.plugins.plugins.notedraw1.api;
```

For convenience, it is also exposed while the plugin is loaded:

```js
const api = window.NoteDraw1;
```

Stable integration API (recommended for Cancip, scripts, and AI plugins):

```js
const noteDraw1 = api.v1;

noteDraw1.apiVersion;       // "1.0"
noteDraw1.capabilities;
noteDraw1.listSurfaces();
await noteDraw1.getState();
await noteDraw1.activate({ tool: "edit-md" });
await noteDraw1.toggle();
await noteDraw1.setVisibility(true);
noteDraw1.setTool("pen");
noteDraw1.setTool({ tool: "draw", brush: "pen", variant: "fountain" });
noteDraw1.setTool({ tool: "draw", brush: "watercolor", variant: "text-highlight" });
noteDraw1.setBrush({ brush: "pen", variant: "fountain", color: "#e53935", width: 4, opacity: 0.9 });
noteDraw1.setTextPreset("rectangle");
noteDraw1.setZoom(1.25);
await noteDraw1.selectElements({ ids: ["element-id"] });
await noteDraw1.updateElements({ ids: ["element-id"], patch: { color: "#43a047", locked: true } });
await noteDraw1.reorderElements({ ids: ["element-id"], direction: "front" });
await noteDraw1.undo();
await noteDraw1.readDrawings("Notes/example.md");
const bundle = await noteDraw1.drawingData.read("Notes/example.md", {
  includeResources: true,
  includeMarkdownLinks: true
});
const hiddenBlock = await noteDraw1.drawingData.serialize(bundle, { format: "block" });
const restoredBundle = await noteDraw1.drawingData.parse(hiddenBlock);
await noteDraw1.writeDrawings("Notes/example.md", drawingData);
await noteDraw1.replaceText({
  path: "Notes/example.md",
  originalText: "old rendered text",
  editedText: "edited **Markdown** text"
});
await noteDraw1.insertStroke("Notes/example.md", stroke);
const unsubscribe = noteDraw1.on("markdown-changed", (event) => console.log(event));
await noteDraw1.execute("set-tool", { tool: "select" });
```

`drawingData.read/parse/serialize` is the placement-neutral integration API for temporary exports. It only returns data and does not create, modify, move, or delete Vault files or change NoteDraw1's storage setting. The calling plugin decides whether to embed the serialized block in a Markdown copy, save JSON beside a note, use a subfolder, or keep the result in memory. `parse` reads standard bundle objects, JSON, hidden blocks, and complete Markdown containing a hidden block. The older `writeDrawings` method remains available separately for integrations that explicitly intend to change NoteDraw1 data.

Custom views from Cancip, NoteWeb, or another plugin can register their visible surface without copying NoteDraw1 internals:

~~~js
const surface = noteDraw1.registerSurface({
  owner: "noteweb",
  id: "tab:" + leaf.id,
  host: webPageContainer,
  source: {
    kind: "url",
    url: currentUrl,
    title: documentTitle
  },
  capabilities: {
    drawing: true,
    textEditing: true,
    elements: true,
    attachments: true
  },
  viewport: {
    getZoom: () => browserZoom,
    setZoom: (zoom) => setBrowserZoom(zoom),
    getScroll: () => ({ left: webPageContainer.scrollLeft, top: webPageContainer.scrollTop }),
    onChange: (callback) => subscribeToViewport(callback)
  }
});

await surface.ready;
await surface.activate("draw");
await surface.execute([
  { op: "set-brush", brush: "watercolor", variant: "text-highlight", color: "#facc15" },
  { op: "insert-elements", elements: generatedStrokes }
]);

surface.destroy();
~~~

Registering the same owner and id again updates its source on the same host. Vault paths, normalized URLs, and virtual keys receive separate persistent drawing storage. The returned handle exposes surface-scoped drawing, text, element, clipboard, history, mind-map, zoom, event, refresh, and lifecycle operations without exposing NoteDraw1's internal controller.

The structured methods cover surface state, visibility, tools, brush settings, text presets, element selection and mutation, layers, locking, note flow, history, clipboard, settings, imports, exports, and lifecycle events. `execute(action, options)` accepts a string, one structured action object, or an ordered action array, so Cancip and other agents do not depend on internal controller method names.

Magic-wand actions are maintained on Markdown, Canvas, PDF, image, Base/database, HTML/webview, and other supported main-workspace pages. State-backed plugin pages use a stable NoteDraw1 storage identity even when they do not expose a Vault file.

Obsidian's built-in global and local graph views remain native interactive surfaces: NoteDraw1 does not automatically wrap their canvases or input handlers. Plugins can still opt a custom region into NoteDraw1 explicitly through `api.v1.registerSurface()`.

The original top-level methods remain available for compatibility.

Example: read current note drawings.

```js
const file = app.workspace.getActiveFile();
const drawings = await app.plugins.plugins.notedraw1.api.v1.readDrawings(file);
console.log(drawings.strokes.length);
```

Example: insert a stroke.

```js
const file = app.workspace.getActiveFile();
await app.plugins.plugins.notedraw1.api.v1.insertStroke(file, {
  brush: "pen",
  color: "#e53935",
  width: 3,
  opacity: 1,
  points: [
    { x: 0.2, y: 0.2 },
    { x: 0.5, y: 0.35 },
    { x: 0.7, y: 0.6 }
  ]
});
```

Example: AI-assisted text replacement.

```js
const file = app.workspace.getActiveFile();
await app.plugins.plugins.notedraw1.api.v1.replaceText({
  file,
  originalText: "old rendered text",
  editedText: "edited Markdown text"
});
```

## AI Editing

The API is intentionally plain JSON and string based so local AI agents can:

- Read drawing layers.
- Insert generated marks, highlights, or review strokes.
- Replace selected or matched text.
- Build higher-level commands such as summarize, rewrite, annotate, or highlight.

For safety, AI tools should read first, prepare a small patch, then write only the target note or drawing file.

## Web Surface Direction

NoteDraw1 is structured around controllers bound to visible note surfaces. That makes future support practical for:

- Obsidian reading view.
- Obsidian source/edit view.
- Obsidian Publish or web-like rendered note pages.
- External AI or browser automation that talks through the public API.

The current package focuses on the local Obsidian plugin runtime. The API and DOM controller split are the extension points for broader web support.

## Version

Current version: `3.5.4`.
