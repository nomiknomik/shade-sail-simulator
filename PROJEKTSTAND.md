# Sonnensegel-Simulator – Projektstand für Weiterarbeit

## Was ist das?
Standalone-Webanwendung (`index.html`, ~966 Zeilen) — kein Build, kein Server nötig,
per Doppelklick in Chrome/Safari öffnen (Internet für CDN-Libs nötig).

## Technischer Stack
- **Three.js 0.160** (3D-Rendering, Schattenwurf via DirectionalLight + ShadowMap)
- **SunCalc 1.9** (Sonnenstand aus Standort/Datum/Uhrzeit)
- **Vanilla JS, ES-Module** (kein Framework, kein Build-Tool)
- Speicherung: **localStorage** (primär) + JSON-Export/Import
- localStorage-Schlüssel: `sonnensegel-v7`

## GitHub-Repository

- **Repo:** https://github.com/nomiknomik/shade-sail-simulator
- **Live-App (GitHub Pages):** https://nomiknomik.github.io/shade-sail-simulator/
- **Branch:** `main`
- **Dateien im Repo:** `index.html`, `README.md` (englisch), `DEVELOPER.md` (technische Doku), `PROJEKTSTAND.md`, `.gitignore`
- **GitHub-Username:** `nomiknomik`
- **PAT-Scope:** `repo`

### Neue Version auf GitHub publizieren — Anweisung für Claude

> Claude hat zwischen verschiedenen Chats keinen persistenten Dateizugriff.
> Das Update läuft vollständig über die **GitHub REST API** (kein lokales Git nötig).

#### Schritt 1 — SHA der Datei abrufen
```bash
curl -s -H "Authorization: token DEIN_PAT" \
  https://api.github.com/repos/nomiknomik/shade-sail-simulator/contents/index.html \
  | python3 -c "import sys,json; print(json.load(sys.stdin)['sha'])"
```

#### Schritt 2 — Datei base64-kodieren
```bash
base64 -w 0 /home/claude/index.html
```

#### Schritt 3 — Datei per API aktualisieren
```bash
curl -s -X PUT \
  -H "Authorization: token DEIN_PAT" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/nomiknomik/shade-sail-simulator/contents/index.html \
  -d "{\"message\": \"vX.Y — Beschreibung\", \"content\": \"BASE64\", \"sha\": \"SHA\", \"branch\": \"main\"}" \
  | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('commit',{}).get('html_url','Fehler:'), d.get('message',''))"
```

Für jede weitere Datei (README.md, DEVELOPER.md, PROJEKTSTAND.md) denselben
SHA-abrufen → base64 → PUT-Ablauf wiederholen.

GitHub Pages aktualisiert sich automatisch innerhalb ~1 Minute.

---

## Aktuelle Version: 3.8 (Datei: index.html)

## Versionshistorie

| Version | Wesentliche Änderungen |
|---------|------------------------|
| 3.3 | Grundversion: Garten, 6 Pfosten, 2 Segel, Möbel, Horn-Quaternion-Best-Fit, Drag, localStorage |
| 3.4 | Laser-Modus entfernt; Bezugspunkt fest auf Ecke 3; Label-Toggle-Checkbox; Segel-Maße als Zahleneingabe; Pergola-Möbel; Möbel-Drag; Segelphysik gefixt |
| 3.5 | Erst-Veröffentlichung auf GitHub; README.md + DEVELOPER.md hinzugefügt |
| 3.6 | 4 Wandhöhen einzeln einstellbar; Eck-Drehpunkt (Pivot) für Tisch + Pergola; Ortsauswahlliste (Freudenstadt, Stuttgart, München, Berlin, Wien, Zürich); Tagesauswertung (10–18 Uhr, m²·h) |
| 3.8 | Schattenfläche auf Gartengrenzen zugeschnitten (Sutherland-Hodgman); Tagesauswertungs-HUD im 3D-View (oben links) |
| 3.7 | Mehrsprachige UI: DE / EN / RU; Label-Schrift 13 px (war 12 px), Labelbreite 96 px |

---

## Alle umgesetzten Features

### Garten
- Breite × Tiefe (Default 9,5 × 9,5 m), **4 Wände einzeln einstellbar** (x0 / xb / y0 / yt, 0 m = keine Wand)
- Nordrichtungs-Regler (Default 229°)

### Mehrsprachigkeit
- Sprachauswahl: 🇩🇪 Deutsch / 🇬🇧 English / 🇷🇺 Русский
- Alle Labels, Buttons, Hinweise, dynamische Ausgaben (Schatten, Sonne, HUD) übersetzt
- Sprachauswahl im localStorage (`shade-lang`), Default Deutsch
- Label-Schriftgröße 13 px, Labelbreite 96 px

### Standort & Zeit
- **Ortsauswahlliste**: Freudenstadt, Stuttgart, München, Berlin, Wien, Zürich, Manuell
- Lat/Lon editierbar
- Datum + Uhrzeit (Schieber + Zeitfeld synchron)
- Tag abspielen (Animation) + Jetzt-Button

### Bezugspunkt
- Fest auf Ecke 3 (+X/+Y), Versatz 0/0/0 (nicht editierbar, vereinfacht Laser-Messung)

### 6 Pfosten (in 6 Farben)
- Rot, Orange, Gelb, Grün, Blau, Violett
- Eingabe: X/Y relativ zum Bezugspunkt, Höhe H absolut ab Boden
- 3D-Labels mit Pfostenbezeichnung und Koordinaten (ein-/ausblendbar per Checkbox)
- Default-Positionen = echte Gartenmessungen

### 2 Segel (je unabhängig)
- 3 oder 4 Ecken wählbar
- Feste Größe: Dreieck → Seiten a/b/c; Rechteck → Breite × Länge
- **Lage aus Pfosten** (Horn-Quaternion-Best-Fit): jede Ecke zeigt gleichwertig zu ihrem Pfosten, Neigung ergibt sich aus Höhenunterschieden
- Verschieben mit Maus (horizontal)
- Drehen per ⟲/⟳-Button (Ecken-Zuweisung rotieren)
- Button „Mitte": setzt Maus-Versatz zurück
- Farbe + Deckkraft frei wählbar
- Seillänge je Ecke: berechnete Ausgabe (kein Input)

### Schatten & Tagesauswertung
- Sonnenstand live aus SunCalc (astronomisch korrekt)
- Echter Schattenwurf via Three.js ShadowMap
- Schattenfläche je Segel: geometrische Projektion + m²-Ausgabe
- Möbel-Schattentest: „im Schatten" / „in der Sonne"
- **Tagesauswertung**: integrierte Bodenschattenfläche 10–18 Uhr in m²·h pro Segel + Gesamt

### Möbel
- Tisch (Breite/Länge/Höhe einstellbar), 2 Stühle, **Pergola** (Breite/Länge/Höhe, wirft Schatten)
- Je: Position X/Y, **Pivot-Drehung** (Drehung um Eck, nicht Mittelpunkt), sichtbar/unsichtbar
- **Drag in XY-Ebene** per Maus im 3D-Fenster

### UI-Details
- Jeder Schieber hat Zahleneingabe daneben (bidirektional synchron)
- Doppelklick auf Schieber → Standardwert
- Export/Import/Reset als JSON
- Auto-Speichern nach jeder Änderung (localStorage `sonnensegel-v7`)

---

## State-Schema (S-Objekt)

```javascript
S = {
  _pivotCorner: true,
  garden: { w, d, walls:{ x0, xb, y0, yt }, north },
  loc:    { lat, lon },
  dateStr: "YYYY-MM-DD",
  timeMin: 0–1439,
  showLabels: boolean,
  ref:    { corner:2, ox:0, oy:0, oz:0 },   // fest
  points: [ { x, y, h } × 6 ],
  sails:  [ {
    on, shape,          // 3 oder 4
    a, b, c,            // Dreieck-Seiten
    bw, bl,             // Rechteck
    dx, dz,             // Maus-Versatz
    posts: [0–5 × shape],
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

---

## Bekannte Einschränkungen

- Segel ist flaches, starres Panel (kein Durchhang — bewusst so)
- Schattenfläche wird am Gartenrand abgeschnitten (Sutherland-Hodgman, seit v3.8) ✓
- Möbel-Schattentest nur punktförmig (nicht gesamte Oberfläche)
- Segel-Drag nur horizontal (keine Höhenänderung per Maus)

## Geplante Features (noch nicht umgesetzt)

1. **Pfosten mit Maus ziehen** (analog zum Segel-Drag)
2. **Dreiecks-Validierung**: Warnung wenn a+b < c
3. Warnung bei zu großem Seil-Restabstand

## CDN-Abhängigkeiten

```
https://unpkg.com/suncalc@1.9.0/suncalc.js
https://unpkg.com/three@0.160.0/build/three.module.js
https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js
```

## Default-Werte

```
Garten: 9,5 × 9,5 m, alle 4 Wände 3 m, Norden 229°
Bezugspunkt: Ecke 3, Versatz 0/0/0
Pfosten 1 🔴  X=  0,0  Y=  0,0  H=2,9 m
Pfosten 2 🟠  X= −7,0  Y=  0,0  H=3,0 m
Pfosten 3 🟡  X= −9,5  Y=  0,0  H=3,0 m
Pfosten 4 🟢  X= −6,2  Y= −9,5  H=2,5 m
Pfosten 5 🔵  X=  0,0  Y= −9,5  H=2,5 m
Pfosten 6 🟣  X=  2,5  Y=  7,5  H=3,0 m
Segel 1: Dreieck, a=6,4 / b=8 / c=4 m, Pfosten 4-5-1 (rot)
Segel 2: aus (Dreieck 3/3/3 m, blau)
```
