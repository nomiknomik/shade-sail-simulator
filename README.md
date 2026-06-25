<div align="center">

# 🌞 Sonnensegel-Simulator

**Browserbasierter 3D-Planer für Sonnensegel / Shade Sails**  
Visualisiert Schattenwurf, Segelausrichtung und Möbel-Beschattung zu jeder Tages- und Jahreszeit.

[![Live Demo](https://img.shields.io/badge/▶%20Live%20Demo-GitHub%20Pages-blue?style=for-the-badge)](https://nomiknomik.github.io/shade-sail-simulator/)
[![Keine Installation](https://img.shields.io/badge/Keine%20Installation-open%20index.html-brightgreen?style=for-the-badge)](#schnellstart)
[![Three.js 0.160](https://img.shields.io/badge/Three.js-0.160-black?style=for-the-badge&logo=threedotjs)](https://threejs.org)

</div>

---

## 📸 Screenshots

<div align="center">

| 3D-Ansicht mit Schattenwurf | Steuerungspanel |
|:---:|:---:|
| ![3D-Ansicht](https://nomiknomik.github.io/shade-sail-simulator/screenshots/3d-view.png) | ![Steuerung](https://nomiknomik.github.io/shade-sail-simulator/screenshots/panel.png) |

> *Screenshot: Sonnensegel über Gartenmöbeln, Sommer 15 Uhr, Freudenstadt*

</div>

---

## ✨ Features auf einen Blick

| | |
|---|---|
| 🏡 **Realer Garten** | Maße, Wandhöhen, Nordausrichtung frei einstellbar |
| ☀️ **Echter Sonnenstand** | Astronomisch korrekt via SunCalc — Ort, Datum, Uhrzeit |
| 🪵 **6 Fixierpfosten** | Direkte X/Y-Eingabe **oder** Laser-Messung (Entfernung + Winkel) |
| ⛵ **Bis zu 2 Segel** | Dreieck oder Rechteck, beliebige Größe und Farbe |
| 🔵 **Auto-Ausrichtung** | Horn-Quaternion-Best-Fit: Segel hängt automatisch korrekt zwischen Pfosten |
| 📐 **Schattenfläche** | Live-Berechnung in m² pro Segel |
| 🪑 **Möbel-Test** | Tisch + Stühle: zeigt ob sie im Schatten oder in der Sonne liegen |
| 🎬 **Tag abspielen** | Animiert Schattenverlauf über 24 Stunden |
| 💾 **Auto-Speichern** | localStorage + JSON-Export/Import |

---

## 🚀 Schnellstart

```bash
# Option A: direkt öffnen (Chrome empfohlen)
open index.html

# Option B: lokaler Server (bei file://-Problemen in Safari)
python3 -m http.server 8080
# → Browser: http://localhost:8080
```

> **Kein Build-Tool, kein npm, keine Installation.**  
> Einzige Voraussetzung: Internetverbindung beim ersten Öffnen (CDN-Bibliotheken).

### 🌐 Online nutzen

**→ [nomiknomik.github.io/shade-sail-simulator](https://nomiknomik.github.io/shade-sail-simulator/)**

---

## 🗺️ Wie es funktioniert

```
┌─────────────────────────────────────────────────────┐
│                 index.html  (alles in einer Datei)  │
│                                                     │
│  ┌─────────────┐    ┌──────────────────────────┐   │
│  │ Steuerungs- │    │     Three.js 3D-Szene     │   │
│  │   panel     │───▶│  Garten · Pfosten · Segel │   │
│  │  (linke     │    │  Möbel · Schatten         │   │
│  │   Seite)    │    │                           │   │
│  └─────────────┘    └──────────────┬────────────┘   │
│                                    │                │
│                         ┌──────────▼─────────┐      │
│                         │  SunCalc-Bibliothek│      │
│                         │  Azimut + Elevation│      │
│                         │  (Echtzeit)        │      │
│                         └────────────────────┘      │
└─────────────────────────────────────────────────────┘
```

---

## 🎮 Steuerung der 3D-Ansicht

| Aktion | Eingabe |
|---|---|
| 🔄 Kamera drehen | `Linke Maustaste` + ziehen |
| 🔍 Zoomen | Mausrad |
| ↔️ Kamera verschieben | `Rechte Maustaste` + ziehen |
| ✋ **Segel verschieben** | `Linke Maustaste` auf Segel + ziehen |
| ⟲ ⟳ Ecken rotieren | Buttons im Pfosten-Panel |
| 🔁 Doppelklick auf Schieber | Zurück auf Standardwert |

---

## 🪵 Pfosten-Eingabe: Direkt vs. Laser

Der Simulator unterstützt zwei Eingabemodi — optimal für Messungen mit dem Laser-Entfernungsmesser:

```
Direkt-Modus          Laser/Polar-Modus
─────────────         ─────────────────────
X   (m)               Entfernung d (m)
Y   (m)               Horizontalwinkel α (°)
H   (m)               Vertikalwinkel β (°)
```

---

## 🏗️ Technischer Stack

| Bibliothek | Version | Zweck |
|---|---|---|
| [Three.js](https://threejs.org) | 0.160 | 3D-Rendering, PCFSoft Shadow Maps |
| [SunCalc](https://github.com/mourner/suncalc) | 1.9 | Astronomischer Sonnenstand |
| Vanilla JS | ES2022 | Kein Framework, kein Build-Tool |

**Algorithmus:** Horn-Quaternion-Best-Fit für die automatische Segel-Orientierung  
(minimiert Fehler zwischen Pfosten-Positionen und Segel-Ecken)

---

## 📁 Projektstruktur

```
shade-sail-simulator/
├── index.html         ← komplette App (~1 Datei)
├── README.md          ← diese Datei
└── PROJEKTSTAND.md    ← technische Zusammenfassung für Weiterentwicklung
```

---

## 🗓️ Default-Konfiguration (eigene Gartenmessungen)

```
Garten:      9,5 × 9,5 m  ·  Wandhöhe 3 m  ·  Norden 229°
Standort:    Freudenstadt, DE  (48,4636° N / 8,4111° O)

Pfosten 1 🔴  X=  0,0  Y=  0,0  H=2,9 m
Pfosten 2 🟠  X= −7,0  Y=  0,0  H=3,0 m
Pfosten 3 🟡  X= −9,5  Y=  0,0  H=3,0 m
Pfosten 4 🟢  X= −6,2  Y= −9,5  H=2,5 m
Pfosten 5 🔵  X=  0,0  Y= −9,5  H=2,5 m
Pfosten 6 🟣  X=  2,5  Y=  7,5  H=3,0 m

Segel 1:  Dreieck  6,4 / 8,0 / 4,0 m  ·  Pfosten 4–5–1
Segel 2:  (deaktiviert)
```

---

## 🔮 Geplante Erweiterungen

- [ ] **Sonnenstunden-Analyse** — für jeden Sitzplatz: wann und wie lange Schatten
- [ ] **Pfosten per Maus ziehen** (analog zu Segel-Drag)
- [ ] **Dreiecks-Validierung** — Warnung bei ungültigen Seitenmaßen
- [ ] **Großer Seilabstand** — Warnung wenn Segel deutlich kleiner als Pfosten-Abstand
- [ ] Weitere Möbel / Objekte (Sonnenschirm, Pflanzenkübel)
- [ ] Hindernisse (Haus, Baum, Mauer)

---

## ⚠️ Bekannte Einschränkungen

- Segel ist ein **flaches, starres Panel** — kein Durchhang/Katenar (bewusst)
- Schattenfläche wird nicht am Gartenrand abgeschnitten (flache Sonne → großer Wert)
- Möbel-Schattentest misst nur einen Punkt pro Objekt (nicht die gesamte Fläche)
- Segel-Drag nur horizontal (keine Höhenänderung per Maus)

---

## 📄 Lizenz

Privates Projekt. Kein öffentliches Release.

---

<div align="center">

Made with ☀️ in Freudenstadt, Germany

</div>
