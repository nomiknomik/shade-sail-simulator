# Sonnensegel-Simulator – Projektstand für Weiterarbeit

## Was ist das?
Standalone-Webanwendung (`index.html`) — kein Build, kein Server nötig,
per Doppelklick in Chrome/Safari öffnen (Internet für CDN-Libs nötig).

## Technischer Stack
- **Three.js 0.160** (3D-Rendering, Schattenwurf via DirectionalLight + ShadowMap)
- **SunCalc 1.9** (Sonnenstand aus Standort/Datum/Uhrzeit)
- **Vanilla JS, ES-Module** (kein Framework, kein Build-Tool)
- Speicherung: **localStorage** (Schlüssel `sonnensegel-v7`) + Cookie (Fallback)
- Export/Import als JSON-Datei

## Aktuelle Version: 3.6 (Datei: index.html)

## Versionsverlauf (Kurz)
- **3.3** – Ausgangsstand: Laser-/Polar-Eingabe, frei wählbarer Bezugspunkt, Best-Fit-Aufspannung
- **3.4** – Laser entfernt; Bezugspunkt fix (Ecke 3); Pfosten-Labels (Checkbox); Segelmaße als Zahleneingabe; Pergola; Möbel per Maus verschiebbar; Starrkörper-Best-Fit korrigiert
- **3.5** – Neues Segel-Modell: schwimmt in der Pfosten-Spannfläche, feste Maße, Maus-Positionierung mit Begrenzung, Seile = kürzeste Verbindung, gebogene Fläche (Hypar) bei 4 Pfosten; Möbel-Labels (Tisch/Pergola)
- **3.6** – Vier getrennte Wandhöhen; Eck-Drehpunkt für Tisch & Pergola; Ort-Auswahlliste; Export-Dateiname mit Zeitstempel; Tagesauswertung Schattenfläche 10–18 Uhr (m²·h, auch im Export)

## Umgesetzte Features (Stand 3.6)

### Garten
- Breite × Tiefe (Default 9,5 × 9,5 m), Nordrichtungs-Regler (Default 229°)
- **4 Wände mit je eigener Höhe** (`S.garden.walls = {x0, xb, y0, yt}`), jede wirft Schatten, 0 m = keine Wand
  - Benennung nach X/Y-Bezugspunkt: x0/y0 = Wände an Bezugsecke, xb/yt = gegenüber
- Standort editierbar + **Ort-Auswahlliste** (Bad Säckingen, Freudenstadt, Karlsruhe, München, Baden-Baden)

### Bezugspunkt
- Fix auf Ecke 3 (+X/+Y), Versatz 0/0/0; cyan-Marker im 3D

### 6 Pfosten
- Rot/Orange/Gelb/Grün/Blau/Violett; Eingabe X/Y (relativ Bezugspunkt) + Höhe (absolut)
- Optionale 3D-Beschriftung (Checkbox `showLabels`): Nummer, X/Y, Höhe

### 2 Segel (Spannflächen-Modell)
- 3 oder 4 Ecken, **feste Maße** (Zahleneingabe)
- Schwimmt in der von den zugeordneten Pfosten gebildeten Fläche:
  - Ausrichtung per 2D-Best-Fit (`bestAngle2D`), jede Ecke zeigt zu ihrem Pfosten
  - Position per Maus (Drag-Offset `s.dx/s.dz`), auf die Pfosten-Begrenzung geklemmt (`pointInPoly`, Binärsuche)
  - Eckenhöhe folgt der Pfostenfläche (`surfaceHeight`): 3 Pfosten = Ebene (baryzentrisch), 4 Pfosten = bilineare/gebogene Fläche (Newton-Inversion, Hypar-Tesselierung 6×6)
  - Seillänge = kürzeste Verbindung Ecke→Pfosten (berechnete Ausgabe)
- ⟲/⟳ rotiert die Ecken-Pfosten-Zuordnung, „Mitte" zentriert; Farbe + Deckkraft

### Sonne & Schatten
- Sonnenstand live aus SunCalc; echter Schattenwurf (ShadowMap)
- Schattenfläche am Boden je Segel (Polygon + m²)
- Möbel-Schattentest (Tisch/Stühle), Pergola-Dach als zusätzlicher Blocker
- Tag abspielen / Jetzt; Tag/Nacht-Wechsel; HUD unten links

### Tagesauswertung (Punkt v3.6)
- `computeDayShade()` integriert die Bodenschattenfläche je aktivem Segel über 10:00–18:00 (Trapezregel, 15-min-Takt) → **m²·h** + Ø in m²
- Anzeige im Panel „Tagesauswertung" (`renderDayShade`), Ergebnis in `S.shadeAnalysis`
- Wird in `updateAll()` aktualisiert und beim Export frisch berechnet
- Gesamt = Summe der Einzelsegel (Überlappung doppelt gezählt)

### Möbel
- Tisch (B/L/H), 2 Stühle, Pergola (B/L/H)
- Position X/Y, Drehung, sichtbar/unsichtbar; alle werfen Schatten
- **Im 3D mit der Maus verschiebbar**
- **Tisch & Pergola: Eck-Drehpunkt** — Geometrie so verschoben, dass eine Ecke im Gruppen-Ursprung liegt; X/Y = diese Ecke (= Drehpunkt). Stühle: Mitten-Drehpunkt
- Labels (bei `showLabels`) für Pfosten, Tisch, Pergola; Tisch-/Pergola-Label sitzen über der Objektmitte

### UI / Persistenz
- Schieber + Zahlenfeld bidirektional; Doppelklick = Default
- Export → `sonnensegel-setup_JJJJ-MM-TT_hhmm.json` (Zeitstempel); Import + `migrate()`; Reset
- Auto-Speichern (localStorage + Cookie)

## Migration (`migrate`)
Bringt alte Stände aufs aktuelle Schema:
- alte einzelne `garden.wallH` → alle vier Wände (`walls.*`)
- einmalige Umstellung Tisch/Pergola von Mitten- auf Eck-Anker (Flag `_pivotCorner`, idempotent)
- Bezugspunkt fix, Pfosten auf {x,y,h} bereinigt, Pergola/Labels ergänzt

## Bekannte Einschränkungen / offene Punkte
- Segel flach bzw. bilinear gebogen, kein Katenar-Durchhang (bewusst)
- Feste Maße im Grundriss exakt; 3D-Kanten durch Höhenunterschiede minimal länger
- Schattenfläche nicht am Gartenrand abgeschnitten
- Möbel-Schattentest nur ein Messpunkt pro Objekt
- Tagesauswertung-Gesamt ohne Überlappungs-Bereinigung (Summe statt Vereinigung)

## Geplante / nicht umgesetzte Features
- Sonnenstunden-Auswertung pro Sitzplatz (von–bis beschattet)
- Pfosten direkt mit der Maus ziehen
- Warnung bei unmöglichem Dreieck (Dreiecksungleichung)
- Überlappungsfreie Gesamt-Schattenfläche (Raster/Vereinigung)
- Hindernisse (Haus, Baum); Regenschutz-Simulation bleibt ausgeschlossen

## CDN-Abhängigkeiten
```
https://unpkg.com/suncalc@1.9.0/suncalc.js
https://unpkg.com/three@0.160.0/build/three.module.js
https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js
```

## Lokaler Start (falls file:// Probleme)
```bash
cd sonnensegel-simulator
python3 -m http.server 8080
# → http://localhost:8080
```
