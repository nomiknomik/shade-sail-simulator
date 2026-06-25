# ☀️ Shade Sail Simulator

A browser-based 3D simulator for planning shade sails in your garden. Visualizes how shade sails cast shadows at any time of day and year, helping you optimize placement, size, and post configuration.

**No installation required** — just open `index.html` in Chrome or Safari.
Internet connection required on first load (CDN libraries).

---

## Live Demo

> Open `index.html` directly in your browser — no server needed.
> If you encounter `file://` issues in Safari, use the local server option below.

```bash
# Local server (optional)
python3 -m http.server 8080
# → http://localhost:8080
```

---

## Features

### 🏡 Garden
- Adjustable width × depth (default: 9.5 × 9.5 m for a real garden in Freudenstadt, Germany)
- 4 shadow-casting walls with adjustable height
- North direction control (rotate the garden to any compass orientation)
- Adjustable location (latitude/longitude) — default: Freudenstadt, 48.46° N / 8.41° E

### 📍 Reference Point
- Any garden corner selectable as the coordinate origin
- All X/Y inputs (posts, furniture) are relative to this point
- Shown as a cyan marker in the 3D view

### 🪵 6 Fixing Posts
Each post has its own color (red, orange, yellow, green, blue, violet):

| Mode | Input |
|---|---|
| **Direct** | X / Y (relative to reference point) + absolute height above ground |
| **Laser/Polar** | Distance + horizontal angle + vertical angle from reference point |

The laser mode is designed for measurements taken with a laser distance meter directly in the garden.

### ⛵ Up to 2 Shade Sails
Each sail is independently configurable:

| Property | Description |
|---|---|
| **Shape** | 3 corners (triangle: sides a/b/c) or 4 corners (rectangle: width × length) |
| **Position** | Derived automatically from the assigned posts (tensioned ropes) |
| **Rope length** | Calculated and displayed — not a manual input |
| **Move** | Drag horizontally with the mouse in the 3D view |
| **Rotate** | ⟲/⟳ buttons cycle the corner-to-post assignment |
| **Color/Opacity** | Freely adjustable |

Sail orientation (including tilt at unequal post heights) is computed automatically via a direction-normalized Horn quaternion best-fit algorithm — each corner points equally toward its assigned post.

### ☀️ Sun & Shadow
- Sun position from real astronomical data (SunCalc library)
- Real-time shadow casting on ground and walls
- Shadow area on the ground shown as a dark polygon + area in m² per sail
- Furniture shade detection: shows whether table/chairs are in shade or sun
- **Play day**: animates the shadow progress over 24 hours
- **Now**: jumps to current time and today's date

### 🪑 Furniture
- 1 table (adjustable width/length/height)
- 2 chairs
- 1 pergola (adjustable width/length/height, casts shade)
- All furniture: X/Y position, rotation, show/hide toggle
- All cast shadows

### 💾 Data Management
- **Auto-save** after every change (localStorage)
- **Export** as `shade-sail-setup.json`
- **Import** a JSON file
- **Reset** to defaults
- **Double-click** on any slider → resets to default value
- Every slider has a **number field** for precise input

---

## 3D View Controls

| Action | Gesture |
|---|---|
| Rotate camera | Left mouse button + drag |
| Zoom | Mouse wheel |
| Pan camera | Right mouse button + drag |
| **Move sail** | Left click on sail + drag |
| **Move furniture** | Left click on furniture + drag |

---

## Technical Stack

| Library | Version | Purpose |
|---|---|---|
| [Three.js](https://threejs.org) | 0.160 | 3D rendering, shadow casting (PCFSoft ShadowMap) |
| [SunCalc](https://github.com/mourner/suncalc) | 1.9 | Sun position (azimuth + altitude) |
| Vanilla JS | ES2022 | No frameworks, no build tools |

Both libraries are loaded via CDN (`unpkg.com`) — no local installation needed.

---

## Project Structure

```
shade-sail-simulator/
├── index.html        ← complete app (single file)
├── README.md         ← this file
└── DEVELOPER.md      ← technical summary for contributors
```

---

## Default Configuration (real garden measurements)

```
Garden:        9.5 × 9.5 m, wall height 3 m, north 229°
Reference:     Corner 3 (+X/+Y), no offset
Location:      Freudenstadt, Germany (48.4636° N / 8.4111° E)
Time:          18:12

Post 1 (red):    X=  0.0  Y=  0.0  H=2.9 m
Post 2 (orange): X= −7.0  Y=  0.0  H=3.0 m
Post 3 (yellow): X= −9.5  Y=  0.0  H=3.0 m
Post 4 (green):  X= −6.2  Y= −9.5  H=2.5 m
Post 5 (blue):   X=  0.0  Y= −9.5  H=2.5 m
Post 6 (violet): X=  2.5  Y=  7.5  H=3.0 m

Sail 1 (red):  Triangle, sides 6.4 / 8.0 / 4.0 m, posts 4–5–1
Sail 2 (blue): off (triangle 3/3/3 m as template)
```

---

## Known Limitations

- Sail is a **flat, rigid panel** — no catenary sag (intentional)
- Shadow area is not clipped to garden boundaries (very large at low sun angles)
- Furniture shade detection uses only one measurement point per object (not the full surface)
- Mouse drag on sails is horizontal only (no height change via mouse)

---

## Planned Features

- [ ] **Sun hours analysis** — for each seat: from when to when and how many hours of shade per day
- [ ] **Drag posts with mouse** (analogous to sail drag)
- [ ] **Triangle validation** — warning if side lengths don't form a valid triangle
- [ ] **Rope slack warning** — when sail is much smaller than post spacing
- [ ] More furniture / objects (e.g. parasol, plant pots)
- [ ] Obstacles (house, tree, wall)

---

## Background

This simulator was built to plan real shade sails for a specific garden in Freudenstadt, Germany. All default values reflect actual laser distance meter measurements taken in that garden. The coordinate system is relative to a user-defined reference point — matching how measurements are naturally taken with a laser meter in a garden setting.

The sail geometry algorithm uses a **Horn quaternion best-fit** to orient each sail such that every corner points equally toward its assigned post. Sail dimensions are fixed by the user; the position and tilt are derived from the post layout.

---

## License

MIT License — feel free to use and adapt for your own garden planning.
