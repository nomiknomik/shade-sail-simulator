# ToDo – Änderungswünsche (Sammelliste)

> **Workflow:** Hier sammelt Alexander offene Verbesserungs- und Änderungswünsche.
> Bei der **nächsten Projektänderung** werden **alle** hier gelisteten Punkte als
> Änderungswünsche berücksichtigt und umgesetzt. Anschließend wird diese Datei
> **wieder geleert** (nur diese Überschrift + Workflow-Hinweis bleiben stehen).


## Offene Optimierungs-Wünsche (B, C, D) – warten auf JSON-Datei

Aktuelle Version ist **v5.6**. Der Optimierer arbeitet bereits mit Greedy-Platzierung,
polishKPI-Feinschliff, Vereinigungs-KPI und **Überlappungsschutz** (SAT).

**B:** Top-5-Positionen je Segel sammeln → alle Kombinationen (Segel 1 × Segel 2) mit
beiden Segeln aktiv nachjustieren, um komplementäre Beschattung auszureizen.

**C:** B + polishKPI kombinieren – nach Top-K-Suche noch Feinschliff drauflegen.

**D:** Gespeicherte JSON-Setups (Ordner `JSON/`) analysieren – Muster der besten
Positionen → Heuristiken für gezielteren Suchalgorithmus ableiten.

**Vorgehen:** Alexander liefert JSON-Datei → dann Klassifizierung B/C/D + Freigabe,
danach parallele Subagenten nach Schwierigkeit (Haiku/Sonnet/Opus).
