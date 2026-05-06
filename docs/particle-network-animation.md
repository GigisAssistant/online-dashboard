# Particle Network Animation — Technischer Bericht

## Übersicht

Die Hintergrundanimation ist ein **3D-Partikel-Netzwerk** gebaut mit [Three.js](https://threejs.org/). Sie läuft in einem `<canvas>` Element das den gesamten Viewport abdeckt und liegt hinter dem eigentlichen Seiten-Inhalt (`z-index: 0`).

Die Animation existiert in zwei Varianten:
- **Dark-Theme** (`index.html`) — Blaue Partikel auf schwarzem Hintergrund
- **Light-Theme** (`login.html`) — Goldene Partikel auf cremefarbenem Hintergrund

---

## Aufbau — Schicht für Schicht

### 1. Canvas & Renderer

```html
<canvas id="hero-canvas"></canvas>
```

```js
const renderer = new THREE.WebGLRenderer({ canvas, alpha: true, antialias: true });
renderer.setSize(W, H);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
```

- `alpha: true` — transparenter Hintergrund, damit die CSS-Hintergrundfarbe durchscheint
- `antialias: true` — weiche Kanten, keine Pixel-Treppeneffekte
- `devicePixelRatio` max 2 — Retina-Support ohne Performance-Einbruch

---

### 2. Kamera

```js
const cam = new THREE.PerspectiveCamera(60, W/H, 0.1, 1000);
cam.position.z = 16;
```

- **Typ:** Perspektivkamera — Objekte die weiter weg sind wirken kleiner (Tiefenwirkung)
- **FOV:** 60° — normales Sichtfeld, nicht zu weit, nicht zu eng
- **Z-Position:** 16 Einheiten vor dem Ursprung — passt zum SPREAD von 24 (Partikel füllen den sichtbaren Bereich)

---

### 3. Partikel — Punkte im 3D-Raum

```js
const N = 110;      // Anzahl Partikel
const SPREAD = 24;  // Verteilt über 24x24x4 Einheiten
```

**Initialisierung:**
```js
for (let i = 0; i < N; i++) {
  const p = {
    x:  (Math.random() - 0.5) * SPREAD,   // -12 bis +12
    y:  (Math.random() - 0.5) * SPREAD,   // -12 bis +12
    z:  (Math.random() - 0.5) * 4,        // -2 bis +2  ← flache Z-Tiefe
    vx: (Math.random() - 0.5) * 0.013,    // Geschwindigkeit X
    vy: (Math.random() - 0.5) * 0.013,    // Geschwindigkeit Y
  };
}
```

- X und Y: breite Streuung über den Viewport
- Z: nur leicht variiert — erzeugt Tiefeneffekt ohne dass Partikel aus dem Sichtfeld verschwinden
- Keine Z-Bewegung — Partikel bewegen sich nur in der XY-Ebene

**Darstellung:**
```js
new THREE.Points(geo, new THREE.PointsMaterial({
  color: 0x4499ff,   // Blau (dark) / 0xC9A020 Gold (light)
  size: 0.07,        // Punktgröße in 3D-Einheiten
  transparent: true,
  opacity: 0.85
}))
```

---

### 4. Verbindungslinien — das Netzwerk

Das ist der Kern der "Graph"-Optik. Jeden Frame wird geprüft: **welche zwei Partikel sind nah genug aneinander um verbunden zu werden?**

```js
const DIST = 4.5;  // Maximaler Abstand für eine Verbindung

const lp = [];
for (let i = 0; i < N; i++) {
  for (let j = i + 1; j < N; j++) {
    const dx = pts[i].x - pts[j].x;
    const dy = pts[i].y - pts[j].y;
    if (dx*dx + dy*dy < DIST*DIST) {        // Pythagoras (ohne sqrt = schneller)
      lp.push(pts[i].x, pts[i].y, pts[i].z,
              pts[j].x, pts[j].y, pts[j].z);
    }
  }
}
lGeo.setAttribute('position', new THREE.Float32BufferAttribute(lp, 3));
```

- O(n²) Algorithmus — bei N=110 sind das maximal 5.995 Paare pro Frame
- Nur 2D-Distanz (XY) — Z wird für die Verbindungsentscheidung ignoriert, nur für die Liniendarstellung genutzt
- Kein Opacity-Fade nach Distanz — alle Linien gleich stark (kann man als Verbesserung einbauen)

**Darstellung:**
```js
new THREE.LineSegments(lGeo, new THREE.LineBasicMaterial({
  color: 0x0066ff,   // Blau (dark) / 0xD4AE40 Gold (light)
  transparent: true,
  opacity: 0.22      // Sehr dezent (dark) / 0.6 (light — prominenter)
}))
```

---

### 5. Bewegung — Bounce-Physik

```js
function tick() {
  requestAnimationFrame(tick);

  for (let i = 0; i < N; i++) {
    const p = pts[i];
    p.x += p.vx;
    p.y += p.vy;
    if (Math.abs(p.x) > SPREAD/2) p.vx *= -1;  // Wand rechts/links → umkehren
    if (Math.abs(p.y) > SPREAD/2) p.vy *= -1;  // Wand oben/unten → umkehren
    a[i*3] = p.x;
    a[i*3+1] = p.y;
  }
  dGeo.attributes.position.needsUpdate = true;
  // ... Linien neu berechnen
  renderer.render(scene, cam);
}
```

- Jeder Partikel bewegt sich geradlinig mit konstanter Geschwindigkeit
- An den Rändern (`±SPREAD/2 = ±12`) prallt er ab → Richtungsumkehr
- Kein Reibung, keine Beschleunigung — simples Billiard-Verhalten

---

### 6. Mouse-Parallax

```js
let mox = 0, moy = 0;
document.addEventListener('mousemove', e => {
  mox = (e.clientX / W - 0.5) * 3;   // -1.5 bis +1.5
  moy = -(e.clientY / H - 0.5) * 3;  // invertiert (Y-Achse)
});

// Im tick():
cam.position.x += (mox - cam.position.x) * 0.03;  // Lerp — weich
cam.position.y += (moy - cam.position.y) * 0.03;
cam.lookAt(0, 0, 0);
```

- Die **Kamera** bewegt sich, nicht die Partikel
- Lerp-Faktor `0.03` = träge, weiche Bewegung (kein harte Springen)
- Maus-Range → Kamera bewegt sich max ±1.5 Einheiten
- `lookAt(0,0,0)` — Kamera schaut immer auf die Mitte

---

### 7. Resize-Handling

```js
window.addEventListener('resize', () => {
  W = innerWidth; H = innerHeight;
  cam.aspect = W / H;
  cam.updateProjectionMatrix();
  renderer.setSize(W, H);
});
```

- Canvas passt sich an Fenstergrößenänderungen an
- Kamera-Aspect-Ratio wird neu gesetzt damit das Bild nicht verzerrt

---

## Vergleich: Dark vs. Light Variante

| Parameter | Dark (`index.html`) | Light (`login.html`) |
|-----------|--------------------|--------------------|
| Partikelfarbe | `#4499ff` (Blau) | `#C9A020` (Gold) |
| Partikelgröße | `0.07` | `0.09` |
| Partikel-Opacity | `0.85` | `1.0` |
| Linienfarbe | `#0066ff` (Blau) | `#D4AE40` (Gold) |
| Linien-Opacity | `0.22` (subtil) | `0.60` (prominent) |
| Geschwindigkeit | `±0.013` | `±0.0143` (minimal schneller) |

---

## Rebuild-Prompt für Claude Code

```
Baue eine Three.js Partikel-Netzwerk-Animation als Hintergrund für eine Webseite.

Die Animation soll auf einem <canvas id="hero-canvas"> laufen, das absolut positioniert 
den gesamten Viewport abdeckt (position: absolute, top/left/right/bottom: 0, z-index: 0).

TECHNISCHER AUFBAU:

1. SETUP
- Three.js via CDN: https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js
- WebGLRenderer mit alpha: true und antialias: true
- devicePixelRatio max 2
- PerspectiveCamera: FOV 60°, near 0.1, far 1000, z-Position: 16
- Kein Hintergrund — alpha: true damit CSS-Background durchscheint

2. PARTIKEL
- N = 110 Punkte
- Zufällig verteilt: X und Y in [-12, +12], Z in [-2, +2]
- Jeder Punkt hat Geschwindigkeit vx und vy, zufällig in [-0.013, +0.013]
- Darstellung: THREE.Points mit PointsMaterial
  - Dark-Variante: color #4499ff, size 0.07, opacity 0.85
  - Light-Variante: color #C9A020, size 0.09, opacity 1.0
- Partikel-Positionen in einem Float32Array, als BufferGeometry

3. VERBINDUNGSLINIEN
- Jeden Frame: O(n²) Loop über alle Partikel-Paare
- Wenn 2D-Distanz (nur XY) zwischen zwei Punkten < 4.5 Einheiten → Linie zeichnen
- Distanzprüfung ohne sqrt: dx*dx + dy*dy < 4.5*4.5
- Darstellung: THREE.LineSegments mit LineBasicMaterial
  - Dark-Variante: color #0066ff, opacity 0.22
  - Light-Variante: color #D4AE40, opacity 0.60
- lGeo.setAttribute('position', ...) jeden Frame neu setzen

4. BEWEGUNG (im requestAnimationFrame tick)
- Jeden Partikel um seine vx/vy bewegen
- Bounce an den Rändern ±12: wenn |x| > 12 → vx *= -1, wenn |y| > 12 → vy *= -1
- BufferGeometry position array direkt updaten, needsUpdate = true setzen
- Linien-Array neu berechnen und neu setzen

5. MOUSE-PARALLAX
- mousemove Listener: mox = (e.clientX/W - 0.5) * 3, moy = -(e.clientY/H - 0.5) * 3
- Im tick: Kamera-Position per Lerp (Faktor 0.03) zur Zielposition bewegen
- cam.position.x += (mox - cam.position.x) * 0.03
- cam.position.y += (moy - cam.position.y) * 0.03
- cam.lookAt(0, 0, 0) jeden Frame

6. RESIZE
- window resize Listener
- W und H updaten, cam.aspect updaten, cam.updateProjectionMatrix(), renderer.setSize()

CSS für den Canvas:
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  z-index: 0;

Der Canvas liegt in einem Container mit position: relative (oder fixed/absolute).
Alle anderen Inhalte haben position: relative und z-index: 1 oder höher.

Kein externes CSS — alles inline. Alles in einem einzigen <script> Block am Ende des body.
Kein TypeScript, kein Build-System — reines Vanilla JS.
```

---

## Mögliche Erweiterungen

- **Opacity-Fade nach Distanz** — Linien werden schwächer je weiter die Punkte auseinander sind: `opacity = 1 - (dist / DIST)`
- **Mehr Tiefe** — Z-Bewegung aktivieren, dann verändert sich die Netzwerk-Optik über die Zeit
- **Touch-Support** — `touchmove` Event für den Parallax-Effekt auf Mobile
- **Color-Theming via CSS-Variable** — Farben aus `getComputedStyle` lesen statt hardcoden
