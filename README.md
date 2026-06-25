# 🌞 Shade Sail Simulator

A browser-based 3D simulator for planning shade sails in your garden.
Visualizes shadow casting, sail orientation, and furniture shading at any time of day and season.

**No installation required** — open `index.html` with a double-click in Chrome/Safari.
Internet connection required on startup (CDN libraries).

**Current version: 3.6**

---

## Quick Start

```bash
# Option A: open directly (Chrome recommended)
open index.html

# Option B: local server (if Safari has file:// issues)
python3 -m http.server 8080
# → http://localhost:8080
```

---

## Features

### 🏡 Garden
- Freely configurable width × depth (default: 9.5 × 9.5 m)
- **Four walls with individual heights** (0 m = no wall), each casts shadows
- Wall naming based on the X/Y reference point: **X=0 / Y=0** = walls at the reference corner, **X=−W / Y=−D** = opposite walls
- North direction control (the garden can be rotated freely relative to north)
- Configurable location, plus a **location preset list** (Bad Säckingen, Freudenstadt, Karlsruhe, Munich, Baden-Baden) or custom coordinates

### 📍 Reference Point
- Fixed at **corner 3 (+X/+Y)**, offset 0/0/0 — origin for all X/Y inputs (posts, furniture)
- Shown as a cyan marker in the 3D view

### 🪵 6 Mounting Posts
Each post has its own color (red, orange, yellow, green, blue, violet) and is configured via
X / Y (relative to reference point) + height (absolute from ground).
Optional: **3D post labels** (number, X/Y, height) toggled via checkbox.

### ⛵ Up to 2 Sails
Each sail is independently configurable:

| Property | Description |
|---|---|
| **Shape** | 3 corners (triangle: sides a/b/c) or 4 corners (rectangle: width × length) |
| **Dimensions** | Fixed dimensions entered as numbers |
| **Position** | Floats within the span surface defined by the posts; drag with mouse between posts |
| **Boundary** | Cannot exceed the post outline (triangle/quadrilateral) |
| **Height** | Follows the post surface — with 4 posts at different heights, a **curved surface (hypar)** |
| **Rope length** | Shortest connection corner→post, calculated and displayed |
| **Rotation** | ⟲/⟳ buttons rotate the corner-post assignment; "Center" recenters |
| **Color/Opacity** | Freely adjustable |

### ☀️ Sun & Shadow
- Sun position from real astronomical data (SunCalc)
- Real-time shadow casting on ground and walls
- Shadow area on the ground as a dark polygon + area in m² per sail
- Furniture shade test: shows whether table/chairs are in shade or sun (pergola roof counts as shade source)
- **Play day**: animates shadow progression over 24 h · **Now**: jumps to current time/date

### 📊 Daily Shade Analysis (10–18 h)
- **Integrated ground shadow area** per active sail and total over the day, in **m²·h** (plus average in m²)
- Trapezoidal rule at 15-minute intervals, based on the selected date
- Result is also included in the JSON export (`shadeAnalysis`) → allows comparison of different configurations
- Note: "Total" is the sum of individual sails (overlaps are counted twice)

### 🪑 Furniture
- 1 table (width/length/height), 2 chairs, 1 pergola (shade roof on 4 legs, casts shadow)
- All furniture: X/Y position, rotation, show/hide toggle
- **Draggable in the 3D view with the mouse**
- **Table & pergola rotate around a corner** (corner anchor = pivot point; X/Y refers to this corner). Chairs rotate around their center.
- With labels enabled: labels showing position and dimensions for table and pergola

### 💾 Data Management
- **Auto-save** after every change (localStorage + cookie)
- **Export** as `sonnensegel-setup_YYYY-MM-DD_hhmm.json` (filename with timestamp)
- **Import** a JSON file (older saves are automatically migrated)
- **Reset** to defaults
- **Double-click** on any slider → resets to default; every slider also has a number input field

---

## 3D View Controls

| Action | Gesture |
|---|---|
| Rotate camera | Left mouse button + drag |
| Zoom | Mouse wheel |
| Pan camera | Right mouse button + drag |
| **Move sail** | Left mouse button on sail + drag (stays within post boundary) |
| **Move furniture** | Left mouse button on furniture + drag |

---

## Tech Stack

| Library | Version | Purpose |
|---|---|---|
| [Three.js](https://threejs.org) | 0.160 | 3D rendering, shadow casting (PCFSoft ShadowMap) |
| [SunCalc](https://github.com/mourner/suncalc) | 1.9 | Sun position (azimuth + altitude) |
| Vanilla JS | ES2022 | No frameworks, no build tools |

Both libraries are loaded via CDN (`unpkg.com`) — no local installation required.

---

## Project Structure

```
shade-sail-simulator/
├── index.html        ← complete app (single file)
├── README.md         ← this file
└── PROJEKTSTAND.md   ← technical summary for continued development
```

---

## Default Values (from actual garden measurements)

```
Garden:      9.5 × 9.5 m, walls 3 m each, north offset 229°
Reference:   corner 3 (+X/+Y), no offset
Location:    Freudenstadt (48.4636° N / 8.4111° E)

Post 1 (red):    X=  0.0  Y=  0.0  H=2.9 m
Post 2 (orange): X= −7.0  Y=  0.0  H=3.0 m
Post 3 (yellow): X= −9.5  Y=  0.0  H=3.0 m
Post 4 (green):  X= −6.2  Y= −9.5  H=2.5 m
Post 5 (blue):   X=  0.0  Y= −9.5  H=2.5 m
Post 6 (violet): X=  2.5  Y=  7.5  H=3.0 m

Sail 1 (red):  Triangle, sides 6.4 / 8.0 / 4.0 m, attached to posts 4–5–1
Sail 2 (blue): off (triangle 3/3/3 m as template)
```

---

## Known Limitations

- Sail is a flat or (with 4 posts) bilinearly curved panel — no true catenary sag (by design)
- Fixed dimensions are exact in the **floor plan**; actual 3D edge lengths are slightly longer due to height differences (a few percent)
- Shadow area is not clipped at the garden boundary (very large at low sun angles)
- Furniture shade test measures only one point per object
- Daily analysis "Total" sums individual sails (no union → overlaps counted twice)

---

## Planned Features

- [ ] **Per-seat shade hours** — when and for how long each seat is in shade
- [ ] **Drag posts with mouse** (like sail/furniture drag)
- [ ] **Triangle validation** — warning for invalid side lengths
- [ ] **Overlap-free total shadow area** (union via raster)
- [ ] Additional furniture / obstacles (house, tree)

---

## License

Private project. No public release.
