# Developer Documentation — Shade Sail Simulator

Technical reference for contributors and future development. Current version: **3.7**

## Architecture

Single-file web application (`index.html`, ~966 lines). No build tools, no package manager, no server required.

```
index.html
├── <style>        CSS custom properties, layout, dark theme
├── <body>         HTML panel (left) + 3D canvas (right)
├── SunCalc CDN    Sun position calculations
├── importmap      Three.js module resolution
└── <script type="module">
    ├── i18n               (translations DE/EN/RU, setLang, t())
    ├── State management   (defaultState, loadState, saveState, migrate)
    ├── Three.js scene     (renderer, camera, lights, OrbitControls)
    ├── Geometry builders  (buildGarden, buildPosts, buildSails, buildFurniture)
    ├── Sail algorithm     (sailSurface, sailCorners — Horn quaternion best-fit)
    ├── Shadow rendering   (updateShadow — raycaster + DirectionalLight)
    ├── Daily analysis     (computeDayShade, renderDayShade — 10–18h, m²·h)
    ├── UI wiring          (wireStaticUI, buildPointEditors, buildSailEditors, ...)
    └── Mouse drag         (onPointerDown/Move/Up — sails + furniture)
```

## CDN Dependencies

```
https://unpkg.com/suncalc@1.9.0/suncalc.js
https://unpkg.com/three@0.160.0/build/three.module.js
https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js
```

## State Schema

All application state lives in the `S` object, persisted to `localStorage` under key `sonnensegel-v7`.

```javascript
S = {
  _pivotCorner: true,                       // furniture rotation mode flag
  garden: {
    w: 9.5,                                 // garden width (m)
    d: 9.5,                                 // garden depth (m)
    walls: { x0:3, xb:3, y0:3, yt:3 },    // individual wall heights (m); 0 = no wall
    north: 229                              // north offset (degrees)
  },
  loc:     { lat: 48.4636, lon: 8.4111 }, // decimal degrees
  dateStr: "YYYY-MM-DD",
  timeMin: 0–1439,                          // minutes since midnight
  showLabels: false,                        // toggle 3D post labels
  ref: { corner:2, ox:0, oy:0, oz:0 },    // reference point (fixed, corner 3 = +X/+Y)
  points: [                                 // 6 posts, relative to ref
    { x, y, h },   // x/y in meters, h = absolute height above ground
    ...
  ],
  sails: [
    {
      on: true,
      shape: 3,                             // 3 = triangle, 4 = rectangle
      a, b, c,                             // triangle sides (m)
      bw, bl,                              // rectangle width/length (m)
      dx: 0, dz: 0,                        // mouse drag offset (m)
      posts: [0–5, ...],                   // post index per corner (length = shape)
      color: "#c0392b",
      opacity: 1
    },
    ...  // up to 2 sails
  ],
  furniture: {
    table:   { on, x, y, yaw, tw, tl, th },          // yaw in degrees
    chair1:  { on, x, y, yaw },
    chair2:  { on, x, y, yaw },
    pergola: { on, x, y, yaw, pw, pl, ph },          // shadow-casting roof
  }
}
```

## Coordinate System

- Origin: reference point (fixed at garden corner 3, +X/+Y direction)
- X axis: positive toward high-X wall
- Y axis: positive toward high-Y wall  
- Z axis: height above ground (Three.js Y-up internally; external API uses X/Y/H)
- North offset: rotates the entire scene; affects sun direction calculation

## Sail Algorithm (Horn Quaternion Best-Fit)

`sailSurface(sail)` determines sail position and orientation:

1. Compute target positions: each corner's assigned post position
2. Build the sail geometry in canonical form (flat in XZ plane)
3. Apply Horn quaternion best-fit registration:
   - Centers both point sets
   - Computes cross-covariance matrix
   - Extracts optimal rotation quaternion (minimizes RMS corner residuals)
   - Translates to best-fit centroid
4. Result: sail center + orientation quaternion → used by Three.js Mesh

**Key invariant:** sail dimensions (a/b/c or bw/bl) are fixed. Only position and orientation are fitted.

## i18n System

```javascript
const LANG = { de: { 'key': 'Text', ... }, en: { ... }, ru: { ... } };
let currentLang = 'de';
function t(key) { return (LANG[currentLang]||LANG.de)[key] || key; }
function setLang(lang) { currentLang=lang; localStorage.setItem('shade-lang', lang); applyTranslations(); }
function applyTranslations() {
  document.querySelectorAll('[data-i18n]').forEach(el => el.textContent = t(el.dataset.i18n));
  // dynamic readouts (ropes, shadows, dayshade) are re-rendered separately
}
```

HTML elements use `data-i18n="key"` attribute. Dynamic content (rope lengths, shadow areas, daily analysis) is re-rendered on language switch via `updateRopeReadouts()`, `updateShadow()`, `renderDayShade()`.

## Daily Shade Analysis

`computeDayShade()` iterates from 10:00 to 18:00 in 15-minute steps:
- Computes sun position via SunCalc for each step
- Projects sail geometry onto ground plane
- Accumulates shadow area (m²) × time step (h) → result in m²·h

Result is stored in `S.shadeAnalysis` and included in JSON export.

## Migration

`migrate(state)` handles upgrades from older localStorage data:
- `garden.wallH` (single value, pre-v3.6) → `garden.walls: { x0, xb, y0, yt }` (all set to former wallH value)
- Reference point always forced to `corner:2, ox:0, oy:0, oz:0` (no longer user-configurable)

## UI Patterns

- **Bidirectional slider + number input:** `linkId(sliderId, numberId, min, max, callback)`
- **Double-click reset:** panel `dblclick` handler checks `el.dataset.default`
- **Furniture drag vs. sail drag:** distinguished by `drag.type` ('furn' | 'sail')
- **Post labels:** `THREE.Sprite` with canvas-rendered text (13px, 96px wide); toggled via `showLabels`
- **OrbitControls:** disabled during drag (`controls.enabled = false`)

## Known Technical Debt

- Shadow area not clipped to garden boundary (low sun elevations → inflated values)
- Furniture shade test uses single center point per piece (not full geometry)
- No triangle inequality check before fitting (degenerate sails render without warning)
- Line count in this doc may drift; always check `wc -l index.html`

## Planned Features

1. **Draggable posts** — `onPointerDown` needs to detect post mesh and update `S.points[i].x/y`
2. **Triangle validation** — check `a+b>c && b+c>a && a+c>b` before `sailSurface()`, show warning
3. **Rope slack warning** — compare computed rope length vs. post distance
