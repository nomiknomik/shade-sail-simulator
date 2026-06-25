# Sonnensegel-Simulator – Projektstand für Weiterarbeit

## Was ist das?
Standalone-Webanwendung (`index.html`, ~807 Zeilen) — kein Build, kein Server nötig,
per Doppelklick in Chrome/Safari öffnen (Internet für CDN-Libs nötig).

## Technischer Stack
- **Three.js 0.160** (3D-Rendering, Schattenwurf via DirectionalLight + ShadowMap)
- **SunCalc 1.9** (Sonnenstand aus Standort/Datum/Uhrzeit)
- **Vanilla JS, ES-Module** (kein Framework, kein Build-Tool)
- Speicherung: **localStorage** (primär) + Cookie (Fallback)
- Export/Import als JSON-Datei

## Aktuelle Version: 3.8 (Datei: index.html)

## Alle umgesetzten Features

### Garten
- Breite × Tiefe (Default 9,5 × 9,5 m), Wandhöhe (3 m), 4 Wände die Schatten werfen
- Nordrichtungs-Regler (Default 229° = eigene Messung)
- Standort: Freudenstadt (48,4636 / 8,4111), editierbar

### Mehrsprachigkeit (v3.4)
- Sprachauswahl: 🇩🇪 Deutsch / 🇬🇧 English / 🇷🇺 Русский
- Alle Labels, Hinweise, Buttons, dynamische Texte (Schatten, Sonne, HUD) sind übersetzt
- Sprachauswahl wird im localStorage gespeichert (`shade-lang`)
- Default: Deutsch
- Label-Schriftgröße auf 13px erhöht (war 12px)

### Bezugspunkt
- Nullpunkt für alle X/Y-Eingaben (Pfosten + Möbel), gemessen ab wählbarer Gartenecke
- Default: Ecke 3 (+X/+Y), Versatz 0/0/0
- Höhe Z des Bezugspunkts gilt nur für Laser-Modus
- Cyan-Marker im 3D zeigt die Position

### 6 Pfosten (in 6 Farben)
- Rot, Orange, Gelb, Grün, Blau, Violett
- Jeder Pfosten: Modus wählbar
  - **Direkt X/Y/H**: X/Y relativ zum Bezugspunkt, Höhe absolut ab Boden
  - **Laser (Polar)**: Entfernung + Horizontalwinkel + Vertikalwinkel vom Bezugspunkt
- Default-Positionen = echte Gartenmessungen (JSON)

### 2 Segel (je unabhängig)
- 3 oder 4 Ecken wählbar
- **Feste Größe**: 3 Ecken → Seiten a/b/c; 4 Ecken → Breite × Länge
- **Lage aus Pfosten** (gespannte Seile):
  - Richtungsbasierter Best-Fit (Horn-Quaternion-Verfahren), normalisiert
  - Jede Ecke zeigt gleichwertig zu ihrem zugeordneten Pfosten
  - Neigung ergibt sich automatisch aus unterschiedlichen Pfostenhöhen
  - Pro Ecke: Pfosten-Dropdown (farbig) + berechnete Seillänge
- **Verschieben mit der Maus** (horizontal im 3D-Fenster)
- **Drehen per Button** ⟲/⟳
- Button „Mitte": setzt Maus-Versatz zurück
- Farbe + Deckkraft je Segel

### Sonne & Schatten
- Sonnenstand live aus SunCalc (Standort + Datum + Uhrzeit)
- Echter Schattenwurf (Three.js ShadowMap)
- Schattenfläche am Boden: geometrische Projektion + m²-Angabe
- Möbel-Schattentest: Tisch/Stühle = „im Schatten" / „in der Sonne"
- Tag abspielen (Animation) + Jetzt-Button
- HUD unten links: Datum + Uhrzeit + Nacht-Hinweis

### Möbel
- 1 Tisch (Breite/Länge/Höhe einstellbar), 2 Stühle
- Je: Position X/Y, Drehung, sichtbar/unsichtbar

### UI-Details
- Jeder Schieber hat **Zahleneingabe** daneben (bidirektional synchron)
- **Doppelklick auf Schieber** → auf Default zurücksetzen
- Export/Import/Reset
- Auto-Speichern nach jeder Änderung (localStorage-Schlüssel: `sonnensegel-v7`)

## Bekannte Einschränkungen / offene Punkte
- Segel ist flaches, starres Panel (kein Durchhang)
- Möbel-Schattentest nur punktförmig (nicht gesamte Oberfläche)

### v3.8: Schatten-Clipping + HUD-Overlay
- Schattenfläche wird jetzt per Sutherland-Hodgman am Gartenrand abgeschnitten (sowohl Tagesauswertung als auch Echtzeit)
- Tagesauswertung wird als permanentes HUD-Overlay links oben auf dem 3D-Viewport angezeigt
- Aktualisiert sich sofort bei jeder Parameteränderung

## Geplante Features
1. **Sonnenstunden-Auswertung**: wie viele h/Tag ist ein Punkt beschattet
2. **Pfosten mit Maus ziehen** (analog zum Segel-Drag)
3. **Dreiecks-Validierung**: Warnung wenn a+b < c etc.

## CDN-Abhängigkeiten
```
https://unpkg.com/suncalc@1.9.0/suncalc.js
https://unpkg.com/three@0.160.0/build/three.module.js
https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js
```

## Default-Werte
```
Garten: 9,5 × 9,5 m, Wand 3 m, Norden 229°
Bezugspunkt: Ecke 3, Versatz 0/0/0
Pfosten 1: X=0,    Y=0,    H=2,9
Pfosten 2: X=−7,   Y=0,    H=3,0
Pfosten 3: X=−9,5, Y=0,    H=3,0
Pfosten 4: X=−6,2, Y=−9,5, H=2,5
Pfosten 5: X=0,    Y=−9,5, H=2,5
Pfosten 6: X=2,5,  Y=7,5,  H=3,0
Segel 1: Dreieck, a=6,4 / b=8 / c=4 m, Pfosten 4-5-1 (rot)
Segel 2: aus (Dreieck 3/3/3 m, blau)
```
