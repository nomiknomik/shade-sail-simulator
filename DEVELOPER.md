# Developer Documentation — Shade Sail Simulator

Technical reference for contributors and future development.

## Architecture

Single-file web application (`index.html`, ~1716 lines). No build tools, no package manager, no server required.

```
index.html
├── <style>        CSS custom properties, layout
├── <body>         HTML panel + 3D stage
├── SunCalc CDN    Sun position calculations
├── importmap      Three.js module resolution
└── <script type="module">
    ├── State management   (defaultState, loadState, saveState, migrate)
    ├── Three.js scene     (renderer, camera, lights, groups)
    ├── Geometry           (garden, posts, sails, furniture)
    ├── Sail algorithm     (bestAngle2D, sailSurface, sailCorners)
    ├── Shadow rendering   (updateShadow, raycaster)
    ├── UI wiring          (wireStaticUI, buildPointEditors, ...)
    └── Mouse drag         (onPointerDown/Move/Up)
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
  garden:  { w, d, walls: { x0, xb, y0, yt }, wallOpacity, north }, // meters, degrees
  loc:     { lat, lon },                    // decimal degrees
  dateStr: "YYYY-MM-DD",
  timeMin: 0–1439,                          // minutes since midnight
  showPostLabels: boolean,                  // 3D post label visibility
  showDims:       boolean,                  // dimension / position arrows
  ref:     { corner: 0–3, ox, oy, oz },    // reference point
  points:  [ { x, y, h } × 6 ],            // post positions (m), relative to ref
  sails:   [ {
    on, shape: 3|4,                         // active flag, triangle or rectangle
    a, b, c,                                // triangle sides (m)
    bw, bl,                                 // rectangle width/length (m)
    dx, dz,                                 // sail-center offset from post centroid (m)
    rot,                                    // rotation angle (degrees)
    posts: [i, j, k] | [i, j, k, l],       // post index per corner
    color, opacity
  } × 2 ],
  furniture: {
    table:   { on, x, y, yaw, tw, tl, th },
    chair1:  { on, x, y, yaw },
    chair2:  { on, x, y, yaw },
    pergola: { on, x, y, yaw, pw, pl, ph },
  },
  person:   { on, x, y, h },               // draggable person figure
  evalRect: { on, x0, y0, x1, y1 },        // evaluation rectangle (seating zone)
  shadeAnalysis: {                          // last computed daily analysis
    window, date, stepMin,
    perSail: [...],                         // per-sail m²·h
    total_m2h,                              // union total
    total_rect_m2h,                         // union inside evalRect
    overlap_m2h                             // double-covered area (waste)
  },
  _pivotCorner: true                        // migration flag
}
```

> **⚠️ Coordinate note:** `S.points[i].y` maps to Three.js **Z** axis (depth), not Y (height). Use `poleTop(i)` to get world position.

## Coordinate System

- **World space**: Three.js right-handed, Y-up. Garden center at origin.
- **Garden corners**: Corner 0 = (−w/2, 0, −d/2), corner 1 = (+w/2, 0, −d/2), corner 2 = (+w/2, 0, +d/2), corner 3 = (−w/2, 0, +d/2).
- **Reference point** (`refWorld()`): world position of the selected garden corner + offset.
- **Post positions**: X/Y in `S.points` are offsets from the reference point. `poleTop(i)` returns the world position of post top.
- **North rotation**: applied to sun direction and north arrow via `S.garden.north` degrees.

## Sail Placement Algorithm

The sail has **fixed dimensions** (triangle sides or rectangle size). Its position and orientation in 3D are derived from the assigned posts.

### Step 1 — Horizontal orientation (`bestAngle2D`)
Find the single rotation angle θ around the vertical axis that minimizes the angular error between each local corner direction and the corresponding post direction:

```
maximize Σ (local_k · post_dir_k) over θ
→ θ = atan2(Σ cross, Σ dot)
```

### Step 2 — Position clamping (`sailSurface`)
The sail center starts at `G + (dx, dz)` (centroid of posts + mouse offset). If any corner falls outside the convex hull of the posts, the offset is binary-searched back until the sail is fully inside.

### Step 3 — Height interpolation (`surfaceHeight`)
Each corner's Y coordinate is interpolated from the post heights:
- **3 posts**: barycentric interpolation in the triangle.
- **4 posts**: bilinear interpolation (Newton iteration for UV coordinates), with triangle fallback.

### Step 4 — Rope lengths
Rope length = Euclidean distance from sail corner (world position) to assigned post top. Displayed as read-only output.

## Shadow Calculation

Shadow area uses geometric projection: each sail corner is projected onto Y=0 along `sunDir`. The resulting polygon is **clipped to the garden rectangle** using the Sutherland-Hodgman algorithm (`clipPolyToGarden`), then the shoelace formula gives the clipped area. This ensures shadow falling outside the garden (on walls or beyond) is not counted.

The daily analysis (`computeDayShade`) integrates shadow area over 10:00–18:00 in 15-minute steps using the trapezoidal rule. Results are shown both in the side panel and as a live HUD overlay on the 3D viewport.

Furniture shade detection fires a ray from the probe point upward along `sunDir` and checks intersection with sail meshes (and pergola roof).

## UI Patterns

- `link(card, cls, lo, hi, apply)` — wires a slider + number field pair inside a card element.
- `linkId(slId, numId, lo, hi, apply)` — same for top-level elements with fixed IDs.
- `bindNum(card, cls, lo, hi, apply)` — number-only input (no slider), used for sail dimensions.
- `rowHTML(...)` — generates slider+number HTML string.
- `numRowHTML(...)` — generates number-only row HTML string.
- All sliders support double-click → reset to `data-default`.

## Persistence

1. `localStorage.setItem('sonnensegel-v7', JSON.stringify(S))` — primary
2. Cookie fallback (first 3900 chars) — for environments where localStorage is blocked

The `migrate()` function handles upgrading old saved states to the current schema.

## Three.js Groups

| Group | Contents |
|---|---|
| `polesGroup` | Post cylinders + knobs (only posts referenced by active sails) |
| `ropesGroup` | Rope lines (corner → post) |
| `furnGroup` | Table, chairs, pergola meshes |
| `shadowGroup` | Ground shadow polygon + outline |
| `labelsGroup` | Sprite labels + dimension arrows |
| `evalGroup` | Yellow evaluation rectangle mesh |
| `dayShadeGroup` | Daily shade analysis visualization |
| `sailMeshes[0/1]` | The two sail meshes (added directly to scene) |

## Optimizer Architecture (v5.x)

### Two-phase sail optimizer (`_runSailOptimizer`)

**Phase 1 — Free placement:**
- Coarse raster (NX=NY=18, ~0.53 m steps) over garden area
- 15° rotation increments
- Sun-tilt: sail inclined toward mean sun direction (10–18h, `tiltPlane`)
- Hill-climb refinement on best candidate
- Both sail orderings tried; best joint score wins

**Phase 2 — Auto-post placement:**
- Each sail corner projected radially to garden wall (`toWall`)
- Nearby corners clustered (shared posts; no two corners of same sail share a post)
- `S.points` filled to 6

**Regression guard:** real KPI (`computeDayShade`) measured before/after; reverted if no improvement.

**Coarse-to-fine polish (`polishKPI`, v5.2):** after main optimizer, 0.30 → 0.15 → 0.05 m hill-climb on dx/dz of each sail using real KPI. Closes the gap between internal search grid and reported precision.

**Diversification (v5.8):** Top-K best Sail-1 positions are kept; for each, best Sail-2 is found independently. Best (S1, S2) pair by joint KPI wins.

### Union shadow (`computeDayShade`, v5.3)

Every garden grid cell is counted at most once across all sails per time step. `perSail[]` retains per-sail values; `total_m2h` = union. `overlap_m2h` = Σ(perSail) − union = wasted double coverage.

---

## Planned Features

See [README.md](README.md#planned-features) for the user-facing roadmap.

Technical notes on planned work:

- **Post drag**: add post meshes to hit-test list in `onPointerDown`; update `S.points[i].x/y` on move.
- **Triangle validation**: check `a + b > c` etc. before `sailLocalCorners()`; show warning in UI.
- **Rope slack warning**: compare rope length vs. post spacing; flag when slack exceeds threshold.
