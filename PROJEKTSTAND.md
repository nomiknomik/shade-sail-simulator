# Sonnensegel-Simulator – Projektstand für Weiterarbeit

## Was ist das?
Standalone-Webanwendung (`index.html`, ~700 Zeilen) — kein Build, kein Server nötig,
per Doppelklick in Chrome/Safari öffnen (Internet für CDN-Libs nötig).
Internet-Verbindung für Three.js + SunCalc (CDN).

## Technischer Stack
- **Three.js 0.160** (3D-Rendering, Schattenwurf via DirectionalLight + ShadowMap)
- **SunCalc 1.9** (Sonnenstand aus Standort/Datum/Uhrzeit)
- **Vanilla JS, ES-Module** (kein Framework, kein Build-Tool)
- Speicherung: **localStorage** (primär) + Cookie (Fallback)
- Export/Import als JSON-Datei

## Aktuelle Version: 3.5 (Datei: index.html)

## Alle umgesetzten Features

### Garten
- Breite × Tiefe (Default 9,5 × 9,5 m), Wandhöhe (3 m), 4 Wände die Schatten werfen
- Nordrichtungs-Regler (Default 229° = deine Messung)
- Standort: Freudenstadt (48,4636 / 8,4111), editierbar

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
- Default-Positionen = deine echten Gartenmessungen (JSON)

### 2 Segel (je unabhängig)
- 3 oder 4 Ecken wählbar
- **Feste Größe**: 3 Ecken → Seiten a/b/c; 4 Ecken → Breite × Länge
- **Lage aus Pfosten** (gespannte Seile):
  - Richtungsbasierter Best-Fit (Horn-Quaternion-Verfahren), normalisiert
  - Jede Ecke zeigt gleichwertig zu ihrem zugeordneten Pfosten
  - Neigung ergibt sich automatisch aus unterschiedlichen Pfostenhöhen
  - Pro Ecke: Pfosten-Dropdown (farbig) + berechnete Seillänge (Anzeige, nicht Eingabe)
- **Verschieben mit der Maus** (horizontal im 3D-Fenster): Klick aufs Segel + ziehen
- **Drehen per Button** ⟲/⟳: rotiert die Ecken-Pfosten-Zuordnung im/gegen Uhrzeigersinn
- Button „Mitte": setzt Maus-Versatz zurück
- Farbe + Deckkraft je Segel
- Default Segel 1: Dreieck, Seiten 6,4/8/4 m, Pfosten 4-5-1 (= deine Konfiguration)
- Default Segel 2: aus (Dreieck 3/3/3 m)

### Sonne & Schatten
- Sonnenstand live aus SunCalc (Standort + Datum + Uhrzeit)
- Echter Schattenwurf (Three.js ShadowMap)
- **Schattenfläche** am Boden: geometrische Projektion als dunkles Polygon + m²-Angabe
- Pro aktivem Segel separate Schattenflächenanzeige
- Möbel-Schattentest: Tisch/Stühle = „🟦 im Schatten" / „☀️ in der Sonne"
- Tag abspielen (Animation) + Jetzt-Button
- Tag/Nacht-Wechsel (Licht + Hintergrundfarbe)
- HUD unten links: Datum + Uhrzeit + Nacht-Hinweis

### Möbel
- 1 Tisch (Breite/Länge/Höhe einstellbar), 2 Stühle, 1 Pergola (Breite/Länge/Höhe)
- Je: Position X/Y (relativ zu Bezugspunkt), Drehung, sichtbar/unsichtbar
- Werfen Schatten; Pergola-Dach wird beim Möbel-Schattentest als Blocker berücksichtigt
- Möbel und Segel per Maus im 3D-Fenster verschiebbar (X/Y-Felder synchronisieren sich)

### UI-Details
- Jeder Schieber hat **Zahleneingabe** daneben (bidirektional synchron)
- **Doppelklick auf Schieber** → auf Default zurücksetzen
- Uhrzeit: Schieber + echtes Uhrzeitfeld (HH:MM)
- **Labels-Toggle**: Pfosten- und Möbelbeschriftungen im 3D ein-/ausblenden (Koordinaten + Maße)
- **Export** → `sonnensegel-setup.json`
- **Import** → JSON-Datei laden
- **Reset** → alle Defaults
- Auto-Speichern nach jeder Änderung (localStorage + Cookie)
- localStorage-Schlüssel: `sonnensegel-v7`

## Bekannte Einschränkungen / offene Punkte

### Physikalisch
- Segel ist **flaches, starres Panel** (kein Durchhang/Katenar — bewusst ausgeschlossen)
- Kein Winkel-/Windmodell
- Schattenfläche wird nicht am Gartenrand abgeschnitten (bei tiefstehender Sonne → sehr groß)

### UI
- Bei zu großem Segel (Seiten > Pfosten-Abstand) wird das Segel korrekt zwischen den Pfosten zentriert, aber die Seillängen sind dann 0 (Ecken liegen hinter den Pfosten)
- Möbel-Schattentest nutzt nur einen Messpunkt pro Objekt (nicht die gesamte Oberfläche)
- Drag-Segel funktioniert nur horizontal (keine Höhenänderung per Maus)

## Geplante / nicht umgesetzte Features
- Sonnenstunden-Auswertung: wie viele h/Tag ist ein Punkt beschattet
- Pfosten direkt mit der Maus ziehen (nur Segel ist derzeit per Maus verschiebbar)
- Warnung bei unmöglichem Dreieck (Dreiecksungleichung verletzt)
- Warnung bei zu großem Seil-Restabstand
- Katenar-Durchhang (ausdrücklich nicht gewünscht)
- Regenschutz-Simulation (ursprünglich in v4 geplant)
- Hindernisse (Haus, Baum)

## Default-Werte (aus deiner JSON-Messung)
```json
Garten: 9,5 × 9,5 m, Wand 3 m, Norden 229°
Bezugspunkt: Ecke 3, Versatz 0/0/0
Pfosten 1: X=0,    Y=0,    H=2,9
Pfosten 2: X=−7,   Y=0,    H=3,0
Pfosten 3: X=−9,5, Y=0,    H=3,0
Pfosten 4: X=−6,2, Y=−9,5, H=2,5
Pfosten 5: X=0,    Y=−9,5, H=2,5
Pfosten 6: X=2,5,  Y=7,5,  H=3,0
Segel 1: Dreieck, a=6,4 / b=8 / c=4 m, Pfosten 4-5-1 (Farbe: rot)
Segel 2: aus (Dreieck 3/3/3 m, blau)
```

## GitHub-Repository

- **Repo:** https://github.com/nomiknomik/shade-sail-simulator
- **Live-App (GitHub Pages):** https://nomiknomik.github.io/shade-sail-simulator/
- **Branch:** `main`
- **Dateien im Repo:** `index.html`, `README.md` (englisch), `DEVELOPER.md` (technische Doku), `.gitignore`

### Neue Version auf GitHub publizieren

Voraussetzung: Git ist lokal installiert, Repo einmalig geklont:
```bash
git clone https://github.com/nomiknomik/shade-sail-simulator.git
cd shade-sail-simulator
```

Neue Version einspielen und pushen:
```bash
# 1. Neue index.html in den Repo-Ordner kopieren (überschreiben)
cp /pfad/zur/neuen/index.html .

# 2. Versionsnummer in index.html anpassen (im <div class="ver">-Tag)

# 3. Commit + Push
git add index.html
git commit -m "v3.x — kurze Beschreibung der Änderungen"
git push

# GitHub Pages aktualisiert sich automatisch innerhalb ~1 Minute
```

Wenn auch README oder DEVELOPER.md geändert wurden:
```bash
git add index.html README.md DEVELOPER.md
git commit -m "v3.x — ..."
git push
```

**GitHub Personal Access Token** (falls beim Push nach Passwort gefragt):
- Username: `nomiknomik`
- Password: das PAT (nicht das GitHub-Passwort) — unter github.com/settings/tokens erstellen, Scope `repo`

## CDN-Abhängigkeiten (beim Öffnen nötig)
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

## Nächste sinnvolle Schritte (Priorität nach Nutzerwunsch)
1. **Sonnenstunden-Auswertung**: Button „Tagesauswertung" → zeigt für jeden Sitzplatz,
   von wann bis wann und wie viele h er beschattet ist
2. **Pfosten mit Maus ziehen** (analog zum Segel-Drag)
3. **Dreiecks-Validierung**: Warnung wenn a+b < c etc.
4. Weitere Möbel / Objekte
