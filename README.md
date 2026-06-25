# 🌞 Sonnensegel-Simulator

Ein browserbasierter 3D-Simulator zur Planung von Sonnensegeln im Garten.
Zeigt Schattenwurf, Segelausrichtung und Möbel-Beschattung zu jeder Tageszeit und Jahreszeit.

**Keine Installation nötig** — `index.html` per Doppelklick in Chrome/Safari öffnen.
Internetverbindung beim Öffnen nötig (CDN-Bibliotheken).

**Aktuelle Version: 3.6**

---

## Schnellstart

```bash
# Option A: direkt öffnen (Chrome empfohlen)
open index.html

# Option B: lokaler Server (bei file://-Problemen in Safari)
python3 -m http.server 8080
# → http://localhost:8080
```

---

## Was der Simulator kann

### 🏡 Garten
- Frei einstellbare Breite × Tiefe (Default: 9,5 × 9,5 m)
- **Vier Wände mit je eigener Höhe** (0 m = keine Wand), jede wirft Schatten
- Wandbenennung nach dem X/Y-Bezugspunkt: **X=0 / Y=0** = Wände an der Bezugsecke, **X=−B / Y=−T** die gegenüberliegenden
- Nordrichtungs-Regler (der Garten kann beliebig zu Norden gedreht werden)
- Standort einstellbar, plus **Ort-Auswahlliste** (Bad Säckingen, Freudenstadt, Karlsruhe, München, Baden-Baden) oder eigene Koordinaten

### 📍 Bezugspunkt
- Fest auf **Ecke 3 (+X/+Y)**, Versatz 0/0/0 — Nullpunkt für alle X/Y-Eingaben (Pfosten, Möbel)
- Wird als cyan-farbener Marker im 3D angezeigt

### 🪵 6 Fixierungspfosten
Jeder Pfosten hat eine eigene Farbe (rot, orange, gelb, grün, blau, violett) und wird über
X / Y (relativ zum Bezugspunkt) + Höhe (absolut ab Boden) eingegeben.
Optional: **Pfosten-Beschriftungen im 3D** (Nummer, X/Y, Höhe) per Checkbox ein-/ausblendbar.

### ⛵ Bis zu 2 Segel
Jedes Segel ist unabhängig konfigurierbar:

| Eigenschaft | Beschreibung |
|---|---|
| **Form** | 3 Ecken (Dreieck: Seiten a/b/c) oder 4 Ecken (Breite × Länge) |
| **Maße** | Feste Maße als Zahleneingabe |
| **Lage** | Schwimmt in der von den Pfosten gebildeten Spannfläche; mit der Maus zwischen die Pfosten ziehen |
| **Begrenzung** | Kann die Kontur zwischen den Pfosten (Dreieck/Viereck) nicht überschreiten |
| **Höhe** | Folgt der Pfostenfläche — bei 4 Pfosten unterschiedlicher Höhe eine **gebogene Fläche (Hypar)** |
| **Seillänge** | Kürzeste Verbindung Ecke→Pfosten, wird berechnet und angezeigt |
| **Drehen** | ⟲/⟳-Buttons rotieren die Ecken-Pfosten-Zuordnung; „Mitte" zentriert wieder |
| **Farbe/Deckkraft** | Frei einstellbar |

### ☀️ Sonne & Schatten
- Sonnenstand aus echten astronomischen Daten (SunCalc)
- Schattenwurf in Echtzeit auf Boden und Wände
- Schattenfläche am Boden als dunkles Polygon + Flächenangabe in m² pro Segel
- Möbel-Schattentest: zeigt an, ob Tisch/Stühle im Schatten oder in der Sonne sind (Pergola-Dach zählt als Schattenspender)
- **Tag abspielen**: animiert den Schattenverlauf über 24 h · **Jetzt**: aktuelle Zeit/Datum

### 📊 Tagesauswertung (10–18 Uhr)
- Über den Tag **integrierte Bodenschattenfläche** je aktivem Segel und gesamt, in **m²·h** (plus Ø in m²)
- Trapezregel im 15-Minuten-Takt, abhängig vom eingestellten Datum
- Wert steckt auch im JSON-Export (`shadeAnalysis`) → verschiedene Konfigurationen vergleichbar
- Hinweis: „Gesamt" ist die Summe der Einzelsegel (Überlappungen werden doppelt gezählt)

### 🪑 Möbel
- 1 Tisch (Breite/Länge/Höhe), 2 Stühle, 1 Pergola (Schutzdach auf 4 Beinen, wirft Schatten)
- Alle Möbel: Position X/Y, Drehung, ein-/ausblendbar
- **Im 3D mit der Maus verschiebbar**
- **Tisch & Pergola drehen um eine Ecke** (Eck-Anker = Drehpunkt; X/Y bezeichnet diese Ecke). Stühle drehen um ihre Mitte.
- Bei aktiver Beschriftung: Labels mit Position und Maßen auch für Tisch und Pergola

### 💾 Datenverwaltung
- **Auto-Speichern** nach jeder Änderung (localStorage + Cookie)
- **Export** als `sonnensegel-setup_JJJJ-MM-TT_hhmm.json` (Dateiname mit Zeitstempel)
- **Import** einer JSON-Datei (ältere Stände werden automatisch migriert)
- **Reset** auf Standardwerte
- **Doppelklick** auf einen Schieber → Standardwert; jeder Schieber hat ein Zahlenfeld daneben

---

## Bedienung der 3D-Ansicht

| Aktion | Geste |
|---|---|
| Kamera drehen | Linke Maustaste + ziehen |
| Zoomen | Mausrad |
| Kamera verschieben | Rechte Maustaste + ziehen |
| **Segel verschieben** | Linke Maustaste auf Segel + ziehen (bleibt in der Pfosten-Begrenzung) |
| **Möbel verschieben** | Linke Maustaste auf Möbel + ziehen |

---

## Technischer Stack

| Bibliothek | Version | Zweck |
|---|---|---|
| [Three.js](https://threejs.org) | 0.160 | 3D-Rendering, Schattenwurf (PCFSoft ShadowMap) |
| [SunCalc](https://github.com/mourner/suncalc) | 1.9 | Sonnenstand (Azimut + Höhe) |
| Vanilla JS | ES2022 | Keine Frameworks, kein Build-Tool |

Beide Bibliotheken werden per CDN geladen (`unpkg.com`) — keine lokale Installation.

---

## Projektstruktur

```
sonnensegel-simulator/
├── index.html        ← komplette App (eine einzige Datei)
├── README.md         ← diese Datei
└── PROJEKTSTAND.md   ← technische Zusammenfassung für Weiterentwicklung
```

---

## Default-Werte (eigene Gartenmessung)

```
Garten:      9,5 × 9,5 m, Wände je 3 m, Norden 229°
Bezugspunkt: Ecke 3 (+X/+Y), kein Versatz
Standort:    Freudenstadt (48,4636° N / 8,4111° O)

Pfosten 1 (rot):    X=  0,0  Y=  0,0  H=2,9 m
Pfosten 2 (orange): X= −7,0  Y=  0,0  H=3,0 m
Pfosten 3 (gelb):   X= −9,5  Y=  0,0  H=3,0 m
Pfosten 4 (grün):   X= −6,2  Y= −9,5  H=2,5 m
Pfosten 5 (blau):   X=  0,0  Y= −9,5  H=2,5 m
Pfosten 6 (violett):X=  2,5  Y=  7,5  H=3,0 m

Segel 1 (rot):  Dreieck, Seiten 6,4 / 8,0 / 4,0 m, hängt an Pfosten 4–5–1
Segel 2 (blau): aus (Dreieck 3/3/3 m als Vorlage)
```

---

## Bekannte Einschränkungen

- Segel ist ein flaches bzw. (bei 4 Pfosten) bilinear gebogenes Panel — kein echter Katenar-Durchhang (bewusst so)
- Feste Maße werden im **Grundriss** exakt gehalten; durch Höhenunterschiede sind die echten 3D-Kantenlängen minimal länger (wenige Prozent)
- Schattenfläche wird nicht am Gartenrand abgeschnitten (bei flacher Sonne → sehr groß)
- Möbel-Schattentest misst nur einen Punkt pro Objekt
- Tagesauswertung „Gesamt" summiert die Einzelsegel (keine Vereinigung → Überlappung doppelt)

---

## Geplante Erweiterungen

- [ ] **Sonnenstunden-Auswertung pro Sitzplatz** — von wann bis wann beschattet
- [ ] **Pfosten per Maus ziehen** (analog zum Segel-/Möbel-Drag)
- [ ] **Dreiecks-Validierung** — Warnung bei ungültigen Seitenmaßen
- [ ] **Überlappungsfreie Gesamt-Schattenfläche** (Vereinigung per Raster)
- [ ] Weitere Möbel / Hindernisse (Haus, Baum)

---

## Lizenz

Privates Projekt. Kein öffentliches Release.
