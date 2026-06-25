# Shade Sail Simulator – Project Status for Continued Development

## What is this?
Standalone web application (`index.html`) — no build step, no server required.
Open with a double-click in Chrome/Safari (internet connection needed for CDN libraries).

## Tech Stack
- **Three.js 0.160** (3D rendering, shadow casting via DirectionalLight + ShadowMap)
- **SunCalc 1.9** (sun position from location/date/time)
- **Vanilla JS, ES modules** (no framework, no build tool)
- Storage: **localStorage** (key `sonnensegel-v7`) + cookie (fallback)
- Export/import as JSON file

## Current Version: 3.6 (file: index.html)

## Version History (Summary)
- **3.3** – Baseline: laser/polar input, freely selectable reference point, best-fit sail orientation
- **3.4** – Removed laser input; fixed reference point (corner 3); post labels (checkbox); sail dimensions as number inputs; pergola; furniture draggable with mouse; rigid-body best-fit corrected
- **3.5** – New sail model: floats within the post span surface, fixed dimensions, mouse positioning with boundary clamping, ropes = shortest connection, curved surface (hypar) for 4 posts; furniture labels (table/pergola)
- **3.6** – Four separate wall heights; corner pivot for table & pergola; location preset list; export filename with timestamp; daily shade analysis 10–18 h (m²·h, included in export)

## Implemented Features (as of v3.6)

### Garden
- Width × depth (default 9.5 × 9.5 m), north direction control (default 229°)
- **4 walls with individual heights** (`S.garden.walls = {x0, xb, y0, yt}`), each casts shadows, 0 m = no wall
  - Named by X/Y reference: x0/y0 = walls at reference corner, xb/yt = opposite walls
- Location editable + **location preset list** (Bad Säckingen, Freudenstadt, Karlsruhe, Munich, Baden-Baden)

### Reference Point
- Fixed at corner 3 (+X/+Y), offset 0/0/0; cyan marker in 3D view

### 6 Posts
- Red/Orange/Yellow/Green/Blue/Violet; input: X/Y (relative to reference) + height (absolute)
- Optional 3D labels (checkbox `showLabels`): number, X/Y, height

### 2 Sails (Span Surface Model)
- 3 or 4 corners, **fixed dimensions** (number input)
- Floats within the surface defined by the assigned posts:
  - Orientation via 2D best-fit (`bestAngle2D`), each corner points toward its post
  - Position via mouse (drag offset `s.dx/s.dz`), clamped to post boundary (`pointInPoly`, binary search)
  - Corner height follows the post surface (`surfaceHeight`): 3 posts = plane (barycentric), 4 posts = bilinear/curved surface (Newton inversion, hypar tessellation 6×6)
  - Rope length = shortest connection corner→post (calculated output)
- ⟲/⟳ rotates corner-post assignment, "Center" recenters; color + opacity

### Sun & Shadow
- Live sun position from SunCalc; real shadow casting (ShadowMap)
- Ground shadow area per sail (polygon + m²)
- Furniture shade test (table/chairs), pergola roof as additional blocker
- Play day / Now; day/night transition; HUD bottom-left

### Daily Shade Analysis (v3.6)
- `computeDayShade()` integrates ground shadow area per active sail over 10:00–18:00 (trapezoidal rule, 15-min intervals) → **m²·h** + average in m²
- Displayed in the "Daily Analysis" panel section (`renderDayShade`), result stored in `S.shadeAnalysis`
- Updated in `updateAll()` and recalculated fresh on export
- Total = sum of individual sails (overlaps counted twice)

### Furniture
- Table (W/L/H), 2 chairs, pergola (W/L/H)
- X/Y position, rotation, show/hide; all cast shadows
- **Draggable in 3D with mouse**
- **Table & pergola: corner pivot** — geometry shifted so one corner sits at the group origin; X/Y = this corner (= pivot point). Chairs: center pivot
- Labels (when `showLabels` active) for posts, table, pergola; table/pergola labels positioned above object center

### UI / Persistence
- Slider + number field bidirectional; double-click = reset to default
- Export → `sonnensegel-setup_YYYY-MM-DD_hhmm.json` (timestamp); import + `migrate()`; reset
- Auto-save (localStorage + cookie)

## Migration (`migrate`)
Brings old saved states to the current schema:
- old single `garden.wallH` → all four individual wall heights (`walls.*`)
- one-time conversion of table/pergola from center pivot to corner pivot (flag `_pivotCorner`, idempotent)
- Reference point fixed, posts normalized to {x,y,h}, pergola/labels added if missing

## Known Limitations / Open Issues
- Sail is flat or bilinearly curved, no catenary sag (by design)
- Fixed dimensions exact in floor plan; 3D edges slightly longer due to height differences
- Shadow area not clipped at garden boundary
- Furniture shade test uses only one measurement point per object
- Daily analysis total without overlap correction (sum instead of union)

## Planned / Unimplemented Features
- Per-seat shade hours (from–to shaded)
- Drag posts with mouse
- Warning for invalid triangle (triangle inequality)
- Overlap-free total shadow area (raster/union)
- Obstacles (house, tree); rain protection simulation explicitly excluded

## CDN Dependencies
```
https://unpkg.com/suncalc@1.9.0/suncalc.js
https://unpkg.com/three@0.160.0/build/three.module.js
https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js
```

## Local Start (if file:// issues)
```bash
cd shade-sail-simulator
python3 -m http.server 8080
# → http://localhost:8080
```
