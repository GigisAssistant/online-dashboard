# Changelog

## v0.5.0 — 2026-05-02
### Changed
- `customer/index.html` — Mouse-Parallax für alle 6 Sektions-Animationen:
  - Globaler Lerp-Mouse-Listener (`factor 0.04`) shared über alle Sections
  - `ctx.translate(pmx, pmy)` in jedem draw-Loop (save/restore)
  - Stärken: Workflow ×144/64, Sales ×160/96, Revenue ×176/128, ROI ×200/144, Contracts ×112/80, Booking ×224/160
- `customer/index.html` — Scroll-Fade komplett überarbeitet:
  - Hero Knowledge Graph: asymmetrisches Fade — runter bei 0.55×, rauf bei 1.3× Hero-Höhe
  - Alle 6 Section-Animationen: IntersectionObserver ersetzt durch kontinuierliche scroll-proportionale Opacity
  - Einblenden (von unten): smooth über 22% Viewport-Höhe
  - Ausblenden (nach oben): smooth über 22% Viewport-Höhe
  - Hochscrollen: Pre-Fade startet 40% Viewport-Höhe vor Sichtbarkeit

## v0.4.0 — 2026-05-02
### Changed
- `customer/index.html` — Background-Animation-System komplett überarbeitet:
  - Hero Three.js Knowledge Graph faded beim Scrollen sauber aus (Opacity via scroll)
  - 6 sektionsspezifische 2D-Canvas-Animationen (IntersectionObserver fade in/out):
    - **Workflow**: Partikel-Streams links→rechts in 5 Lanes (gleiche DNA wie Knowledge Graph, direktional)
    - **Sales**: 60 Partikel vorverteilt über Sektionshöhe, steigen in 6 Spalten auf — sofortiger Start
    - **Revenue**: 5 überlagerte Sinuswellen mit unterschiedlicher Frequenz & Amplitude
    - **ROI**: 3 Radar-Pulse expandieren von den Karten-Mittelpunkten nach außen
    - **Verträge**: Dot-Grid (42px Raster) mit glühendem Scan-Beam
    - **Terminbuchung**: 3 elliptische Orbitalbahnen mit Kometenschweif-Dots
  - Alle Opacity-Werte auf Level des Knowledge Graphs kalibriert (mehrfach angepasst)
  - `.sec` erhält `overflow:hidden`, `.sec-in` erhält `position:relative;z-index:2`

## v0.3.0 — 2026-05-02
### Added
- `customer/index.html` — Customer Dashboard (vollständig)
  - Three.js Gold Knowledge Graph (Hintergrund Hero, 130 Partikel, Maus-Parallax)
  - n8n-Style Workflow-Visualisierung (5 Nodes, fließende SVG-Pfeile, Spinner-Animation)
  - Vertriebsdashboard: 6 KPI-Karten mit Metric-Bars + Bar-Chart (Nov–Apr)
  - Umsatz & Zahlungen: Animierter SVG-Linienchart (Draw-in), Zahlungsstatus-Liste
  - ROI Dashboard: Automatische Berechnung ab Vertragsbeginn (15.01.2026), Count-up Animationen
  - Verträge: Filterfunktion (Alle/Unterzeichnet/Ausstehend), PDF Download/Upload
  - Terminbuchung: Placeholder für Reclaim.ai iframe
  - GSAP ScrollTrigger für alle Sektionen (fade-up, stagger, count-up, bar-fill)
  - Responsive (Mobile/Tablet Breakpoints)

## v0.2.0 — 2026-05-02
### Added
- `login.html` — einheitliche Login-Seite für Customer + Employee (umbenannt von `customer.html`)
- `customer/` Ordner angelegt — für Customer Dashboard
- `employee/` Ordner angelegt — für Employee Dashboard (Inhalt noch offen, siehe TODO.md)
- Airtable: Login-Felder `password_hash`, `reset_token`, `reset_token_expires` in **employee** und **costomer** Tabelle hinzugefügt

### Changed
- `index.html` — "Customer Login" Button zeigt jetzt auf `login.html`

## v0.1.0 — 2026-05-02
### Added
- Projektstruktur angelegt
- `index.html` — Landing Page mit Three.js Partikel-Hintergrund, ROI-Rechner, i18n (EN/DE)
- `customer.html` — Login-Page (Gold-Theme, Partikel-Hintergrund)
- README und CHANGELOG initialisiert
