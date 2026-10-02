# CLAUDE.md — Heaterium

Single-file vanilla JS app (`index.html`). No framework, no bundler. Keep it dependency-free unless asked.

## Rendering pipeline (the core idea)

1. **Mask**: content (text via canvas `fillText`, or an uploaded image) is rasterised into a white silhouette PNG (`buildTextMask` / `buildUploadMask`), auto-cropped by `cropAlpha`. Rasterising avoids font-embedding problems when the SVG is later drawn into a canvas for export.
2. **Stripe**: `#ht-stripe` is a repeating `linearGradient` (grey → dark → grey) whose `gradientTransform` is translated each frame — that is the animation.
3. **#ht-material** filter: `blur(SourceAlpha) → arithmetic(SA − blur) → overlay with SourceGraphic` gives the bevelled grey shading.
4. **#ht-color** filter: blur → add grain (static noise tile via `feImage` + `feTile`, weighted so the background stays clean) → `feComponentTransfer` table maps grey to the palette.

All filter values live in a local coordinate system (≈180 units per em), so effects scale with the artwork.

## Effect mask

`S.fxType` is `none | linear | radial`. Geometry is stored in artboard pixels (`S.fxL = {x1,y1,x2,y2}`, `S.fxR = {cx,cy,r}`) and converted to the item's local space in `fxGradAttrs(spec)`.
The fade is **not** an SVG `<mask>` on the filtered group. Instead `fxLayers()` paints a background-colored cover rect (and optionally the plain artwork) on top, with gradient `stop-opacity` (`fxGrad`). This keeps rendering identical across browsers and in exports.
On-canvas editing: `setFxEdit`, `drawFxOverlay` (screen-space `#fxOverlay` SVG), pointer modes `fxdraw` and `fxh`; `setFxGeom()` is the fast update path.

## Key functions

- `spec(scale)` — single source of truth for geometry/colors, shared by preview and every export.
- `svgMarkup(spec, phase, {smil, scale})` — builds the full SVG string. `smil:true` adds `<animateTransform>` for the animated SVG export.
- `applyAll()` — full preview rebuild (palette/filter changes). `setGeom()` — fast path for move/scale (only updates transforms and filter regions). `updateStripe()` — per-frame animation.
- `updateView()` — zoom/pan is done by changing the preview `<svg>` `viewBox` (no CSS scaling), so cost stays bounded by the viewport.
- Overlay: `itemBox`, `updateOverlay`, pointer handlers (`mode: move | scale | pan`).
- Export: `drawFrame` (SVG → Image → canvas), `exportVideo` (WebCodecs + `Mp4Muxer`, MediaRecorder fallback), `saveFile`.
- Video codecs: `VCODECS` + `pickEncoder`. H.265 (hvc1) is the default and falls back to H.264. Uses constant-quality `quantizer` mode, with per-pixel VBR as fallback. VP9 is deliberately excluded: browser VP9 encoders run in realtime mode and produced files 4–7× larger on this grainy material.

## State

Everything is in the object `S` (text, font, palette, glow, edge, grain, duration, angle, period, phase, ratio, size, offX, offY, export settings). `DEFAULT_LOOK` holds the material/motion defaults used by Reset.

## Notes

- `saveFile` first tries `window.claude.use('downloads')` (claude.ai artifact runtime); elsewhere it falls back to a normal `<a download>`. Safe to keep or remove.
- Don't use `localStorage` assumptions or external network calls beyond Google Fonts and the jsDelivr fallback.
- Heavy SVG filters: test performance on large canvases after changes.
