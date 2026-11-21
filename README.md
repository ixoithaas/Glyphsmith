# Glyphsmith

Turn an image of a hand-written (or printed) alphabet sheet into a real,
installable **TrueType font (.ttf)** — entirely in the browser. No uploads,
no server, no build step.

## Run it

Just open **`index.html`** in any modern browser (double-click the file, or
serve the folder with e.g. `python3 -m http.server`). It is a single
self-contained page; the only network requests are its two CDN libraries
(Tailwind CSS and opentype.js), after which everything — slicing, tracing,
font assembly, download — happens locally in the tab.

## Workflow

1. **Upload** — drag & drop, browse, or paste a photo/scan of your character
   sheet. (Or press *“Load demo sheet & auto-slice it”* to try the whole
   pipeline on a sheet the app draws for you.)
2. **Map & slice** — draw a box around each character and assign it to a key.
   Extras: 🪄 click-to-select (connected-component picking), **Auto-detect**
   (finds ink blobs, re-unites dotted `i`s/colons, assigns in reading order),
   equal **grid slicing**, pan/zoom canvas, move/resize handles, undo, a live
   character map showing mapped vs. missing glyphs, and per-glyph
   crop → threshold → vector previews with threshold/Otsu, detail,
   speckle clean-up and smoothing controls.
3. **Build & test** — name the font, pick *Smart normalize* sizing (cap
   height / x-height / descenders / punctuation placement per character
   class) or *preserve position in box* for gridded sheets, build, and
   proof it in the live type tester + specimen (the generated font is
   loaded into the page via the FontFace API before you download).
4. **Download** — get a genuine TrueType `.ttf` (sfnt `0x00010000`, glyf
   outlines, 1000 units-per-em) and install it on Windows, macOS or Linux.

## How it works

- Each box is cropped, thresholded to a 1-bit bitmap, and traced with a
  marching-squares contour follower; contours are simplified
  (Douglas–Peucker), optionally smoothed into quadratic Béziers, and holes
  are counter-wound so letters like `o`, `a`, `8` keep their counters.
- Glyphs are built as `opentype.Glyph` paths on a
  `new opentype.Font({...})` (opentype.js 1.3.4, CDN) with per-class
  metrics. Note: that release's `toArrayBuffer()` only writes
  CFF-flavoured OpenType, so Glyphsmith also contains a small serializer
  that walks the same `opentype.Font` object and emits genuine TrueType
  `glyf`/`loca`/`cmap`/`head`/`hhea`/`hmtx`/`maxp`/`OS/2`/`name`/`post`
  tables (verified byte-for-byte with FontTools: checksums, cmap, glyph
  bounds and outlines all validate, and the output re-parses cleanly).
- Everything is vanilla JS + Canvas 2D in one file — the tracing,
  auto-detection (connected components) and TTF writer are implemented
  from scratch in `index.html`.
