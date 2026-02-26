# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

YuFo2026 is a static web application for the Tzu Chi Buddha Bathing Ceremony (慈济浴佛 2026). It has two pages:

- **`public/index.html`** — Registration interface with an interactive totem formation visualizer
- **`statistics/index.html`** — Real-time statistics dashboard fed from Google Sheets

There is no build system, no package manager, and no backend. Everything is vanilla HTML/CSS/JavaScript.

## Running the Project

Serve the `public/` directory with any static HTTP server:

```bash
python3 -m http.server --directory public 8000
# or
npx http-server ./public
```

Open `statistics/index.html` directly (or serve it) to view the statistics dashboard.

## Architecture

### Main Registration Page (`public/index.html`)

- On load, reads `shape.png` via a Canvas element to detect dark pixels, which become interactive "dots" representing person positions in the totem formation (up to 6000 dots, set in `CONFIG`).
- User submits a registration form → a random unlit dot gets "lit" with an animation → count is saved to `localStorage` (`tzuchi_yufo_2026_v1`).
- Bilingual (zh/en): language toggling is handled by reading `data-zh` / `data-en` attributes on elements; preference stored in `localStorage` (`tzuchi_language`).

Key hardcoded config at the top of the script block:
```javascript
const CONFIG = {
    totalDots: 6000,
    targetCount: 6000,
    storageKey: 'tzuchi_yufo_2026_v1',
    imagePath: 'shape.png'
};
```

### Statistics Dashboard (`statistics/index.html`)

- Fetches registration data from a public Google Sheet via the `gviz` CSV export API (no auth required).
- Parses CSV with a custom parser and renders doughnut charts via Chart.js (CDN).
- Auto-refreshes every 60 seconds (`REFRESH_INTERVAL`).
- Key constants near the top of the script: `SPREADSHEET_ID`, `GID`, `REFRESH_INTERVAL`.

## Key Patterns

**Internationalization:** Content uses `data-zh` / `data-en` attributes (and `data-placeholder-zh/en`, `data-label-zh/en` for inputs/selects). A single `setLanguage()` function iterates over these to swap all text.

**Image-based dot layout:** The totem grid is not a fixed grid — it is derived by analysing the pixel brightness of `shape.png` at runtime using `CanvasRenderingContext2D.getImageData()`. Changes to formation layout require updating the image file.

**All CSS is inline** inside `<style>` tags in each HTML file. There are no external stylesheets.

**No external dependencies for `index.html`** — only Google Fonts and the Buddha/shape images are loaded externally. The statistics page additionally uses Chart.js from CDN.
