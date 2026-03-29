# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

An interactive town map viewer for the fictional town of **Willowshore**. It displays a large map image with pan/zoom, clickable markers for locations of interest, static map labels, and a settings panel for filtering markers by tag. No build step — serve with a local HTTP server (required for `fetch()`) and open in a browser.

```
python3 -m http.server 8080
```

## File Structure

- `index.html` — single-file app (HTML + CSS + JS)
- `willowshore.json` — map configuration (image, dimensions, categories, labels, markers)
- `images/townmap.webp` — the map image (5315×3050px)

## Stack

- **HTML/JS** — no framework, no build step
- **Tailwind CSS** — CDN only
- **Font Awesome 6** — CDN, used for marker icons
- **Google Fonts** — Eagle Lake serif font, used for all UI text

## Map Configuration (`willowshore.json`)

All map data lives in the JSON file. `index.html` fetches it on load.

**Top-level fields:**
- `image` — path to the map image
- `width` / `height` — natural pixel dimensions of the image (used as the marker coordinate space)
- `categories` — named category definitions: `{ color, faIcon }`
- `labels` — array of static map labels (see below)
- `markers` — array of location markers (see below)

**Marker fields:** `x`, `y` (image pixel coords), `category`, `title`, `description`, `tags` (array of strings)

**Label fields:** `x`, `y`, `text`, `vertical` (bool), `fontSize` (map pixel units, default 50)

**Categories:** `trade`, `shrine`, `landmark` — each defines `color` (rgba) and `faIcon` (Font Awesome icon name).

**Tags:** `house`, `shop`, `landmark`, `eatery`, `shrine`, `gate` — used for filtering in the settings panel.

## Pan & Zoom

- `scale`, `originX`, `originY` — current transform state
- `baseImgW`, `baseImgH` — display size of the image at scale=1 (computed from natural dimensions + container size); required for correct clamping
- `zoomAt(clientX, clientY, delta)` — zooms toward a point, clamps to `[MIN_SCALE, MAX_SCALE]`
- `clampOrigin()` — prevents panning beyond image edges; uses `baseImgW/H` and flexbox offset `(containerW - baseImgW) / 2` for correct asymmetric bounds
- `applyTransform()` — writes CSS transform to `<img>`, then calls `updateStaticLabels()` and `updateMarkers()`
- Trackpad pinch: `wheel` event with `ctrlKey: true`, speed `0.01`; mouse wheel speed `0.001`

## SVG Overlay

The `#marker-svg` covers the container and has three layers (in render order):

1. `#static-labels-layer` — map labels that scale with the map; transform is updated in `updateStaticLabels()` to mirror the image transform
2. `#markers-layer` — marker icons (FA text) and selected-state circles, grouped per marker in `<g>` elements
3. `#labels-layer` — marker title signboards (always on top of markers)

## Markers

- Defined in `willowshore.json`, loaded via `fetch()`
- `createMarkerEls()` builds SVG elements per marker; `updateMarkers()` recomputes screen positions from image coords
- **Unselected state:** icon visible, circle hidden; **Selected state:** circle animates in, icon fades out, title signboard appears
- Selecting a marker also shows the `#desc-frame` overlay (top-right, fixed) with title + description
- `updateMarkerVisibility()` shows/hides marker groups based on checked tags in the settings panel

## Static Labels

- Created by `createStaticLabels()` from `data.labels`
- Live in `#static-labels-layer`; the group transform is updated on every pan/zoom to mirror the map image transform: `translate(flexL + originX, flexT + originY) scale(scale * baseImgW / mapWidth)`
- Vertical labels stack individual characters top-to-bottom (upright, not rotated)
- `fontSize` is in map pixel units, so labels scale proportionally with zoom

## UI Overlays

Both overlays share the same visual style: notched-corner frames with layered yellow/red borders, Eagle Lake font.

- **Description frame** (`#desc-frame`) — fixed, top-right; shows selected marker's title (bold) and description; inverted palette (yellow bg, red text/border)
- **Settings frame** (`#settings-frame`) — fixed, bottom-left; tag filter checkboxes; minimizable via `−`/`+` toggle

## Key Constants (top of `<script>`)

| Constant | Purpose |
|---|---|
| `MIN_SCALE` | Minimum zoom (1 = fit to viewport) |
| `MAX_SCALE` | Maximum zoom level |
| `ZOOM_SPEED` | Mouse wheel sensitivity |
| `MARKER_COLOR` | Default marker color (rgba) |
| `MARKER_RADIUS` | Marker circle radius in px |
| `FA_ICONS` | Map of Font Awesome icon names → unicode codepoints |
