# Heaterium

A single-page tool that turns text or an uploaded SVG/PNG into an animated "thermal" material, built entirely with SVG filters. The UI follows the shadcn design system (zinc tokens, light/dark).

**[Try it in your browser →](https://genz000.github.io/heaterium/)**

Nothing to install and no sign-up. Everything runs locally in the browser. Chrome, Edge or Safari give the best video export.

## Run it locally

No build step. Either open `index.html` directly in Chrome, Safari or Firefox, or serve the folder:

```bash
npx serve .          # or: python3 -m http.server 8000
```

Serving over http is recommended (clipboard and some download behaviour are stricter on `file://`).

## Features

- Text (12 Google Fonts, weight, tracking, line height) or upload SVG / PNG / WebP
- 6 palettes, invert, heat, glow, edge depth, shading contrast, grain
- Animated light band: loop length, direction, band width, start offset
- Canvas: 9 aspect-ratio presets + custom size up to 4096 px
- Move and scale the artwork on the canvas (handles, snapping, Alt/Shift modifiers, arrow-key nudge) or by exact X / Y / % values
- Effect mask: linear or radial fade drawn on the canvas, with feather, invert, and the option to show plain artwork outside the mask
- Zoom and pan: toolbar, Ctrl/Cmd + scroll, pinch, `+ − 0 1` keys
- Export: PNG (0.5×–3×), self-contained animated SVG, MP4 in H.265 (default) or H.264, encoded frame by frame with WebCodecs at constant quality (High / Balanced / Smallest); WebM fallback for browsers without WebCodecs

## Files

| Path | Purpose |
| --- | --- |
| `index.html` | The whole app: HTML, CSS and JS inline |
| `vendor/mp4-muxer.js` | MP4 muxer (MIT) used for video export; falls back to jsDelivr if missing |
| `CLAUDE.md` | Architecture notes for Claude Code |
