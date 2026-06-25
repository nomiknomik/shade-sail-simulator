# Developer Documentation — Shade Sail Simulator

Technical reference for contributors and future development.

## Architecture

Single-file web application (`index.html`, ~700 lines). No build tools, no package manager, no server required.

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
  garden:  { w, d, wallH, north },       // meters, degrees
  loc:     { lat, lon },                  // decimal degrees
  dateStr: "YYYY-MM-DD",
  timeMin: 0–1439,                        // minutes since midnight
  showLabels: boolean,
  ref:     { corner: 0–3, ox, oy, oz },  // reference point
  points:  [ { x, y, h } × 6 ],          // post positions (m), relative to ref
  sails:   [ {
    on, shape,                            // 3 or 4 corners
    a, b, c,                              // triangle sides (m)
    bw, bl,                               // rectangle width/length (m)
    dx, dz,                               // mouse drag offset (m)
    posts: [0–5 × shape],                 // post index per corner
    color, opacity
  } × 2 ],
  furniture: {
    table:   { on, x, y, yaw, tw, tl, th },
    chair1:  { on, x, y, yaw },
    chair2:  { on, x, y, yaw },
    pergola: { on, x, y, yaw, pw, pl, ph },
  }
}
```

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

Shadow area uses geometric projection: each sail corner is projected onto Y=0 along `sunDir`. The shoelace formula gives the projected polygon area.

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
| `polesGroup` | Post cylinders + knobs |
| `ropesGroup` | Rope lines (corner → post) |
| `furnGroup` | Table, chairs, pergola meshes |
| `shadowGroup` | Ground shadow polygon + outline |
| `labelsGroup` | Sprite labels (shown when `showLabels` is true) |
| `sailMeshes[0/1]` | The two sail meshes (added directly to scene) |

## Planned Features

See [README.md](README.md#planned-features) for the user-facing roadmap.

Technical notes on planned work:

- **Sun hours analysis**: iterate over `timeMin` in steps, call `updateShadow()` logic without rendering, accumulate shaded minutes per furniture probe point.
- **Post drag**: add post meshes to hit-test list in `onPointerDown`; update `S.points[i].x/y` on move.
- **Triangle validation**: check `a + b > c` etc. before `sailLocalCorners()`; show warning in UI.
