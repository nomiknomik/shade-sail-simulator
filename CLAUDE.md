# Shade Sail Simulator – Claude Code Context

## Project Overview
A browser-based 3D shade sail simulator for planning sail installations and shadow analysis.
**Current version:** v4.1  
**Tech stack:** Single `index.html` – Three.js 0.160 + SunCalc 1.9 via CDN, no build tools.

## Repository
- GitHub: `https://github.com/nomiknomik/shade-sail-simulator`
- Live: `https://nomiknomik.github.io/shade-sail-simulator/`

## ⚠️ ToDo-Workflow (PFLICHT bei jeder Projektänderung)
Die Datei `ToDo.md` ist die zentrale Sammelliste für Änderungswünsche von Alexander.

**Vor jeder Projektänderung** (jede inhaltliche Änderung an `index.html` oder am Projekt):
1. **`ToDo.md` zuerst lesen.**
2. Alle dort unter „Offene Wünsche" gelisteten Punkte als zusätzliche Änderungswünsche
   in die aktuelle Aufgabe aufnehmen und mit umsetzen.
3. Nach erfolgreicher Umsetzung **`ToDo.md` leeren** – d. h. nur Überschrift und den
   Workflow-Hinweis stehen lassen, alle erledigten Stichpunkte entfernen.
4. Die geleerte `ToDo.md` zusammen mit den übrigen Änderungen committen/pushen.

So gehen keine Wünsche verloren und die Liste bleibt nach jeder Runde sauber.

## Architecture
Everything lives in `index.html` (≈1100 lines). Key sections:
- `defaultState()` – all garden/sail/furniture state (localStorage key: `sonnensegel-v7`)
- `TRANSLATIONS` object – DE/EN/RU i18n
- `pn()` helper – parses numbers (accepts comma as decimal separator)
- `clipPolyToGarden()` – Sutherland-Hodgman polygon clipping for shadow area
- Sail surface: bilinear/hyperbolic paraboloid model (4-post interpolation)
- Sun position: SunCalc library → directional Three.js light
- `updateFromState()` – rebuilds all 3D objects from state

## Key Design Decisions
- **No build step** – must stay as single file deployable via GitHub Pages
- **State migration** – all migrations must be idempotent (check for flag like `_pivotCorner`)
- **Coordinates** – X/Y relative to selectable reference corner; Z = absolute height
- **Rope lengths** – computed outputs, not inputs; sail floats within post polygon

## Garden Defaults (Alex's garden)
- Size: 9.5 × 9.5 m
- Location: Freudenstadt, Germany (48.4667°N, 8.4167°E)
- North direction: 229°

## GitHub Workflow (via REST API)
After changes, update GitHub with:
```bash
# Fetch SHA → PUT with new content
SHA=$(curl -s -H "Authorization: token TOKEN" \
  "https://api.github.com/repos/nomiknomik/shade-sail-simulator/contents/index.html" \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['sha'])")
```
Commit messages in English: `feat:`, `fix:`, `docs:`, `chore:`

## Planned Features (backlog)
- Sun-hours analysis per seating position
- Mouse-dragging of posts (currently only sails draggable)
- Triangle validity warnings
- Warnings for excessive rope residual lengths
