# TODO

## Offen

### Auth / Backend
- [ ] `server.js` — Node.js + Express Server aufsetzen mit:
  - `POST /auth/login` — Email + Passwort prüfen (bcrypt), JWT zurückgeben
  - `GET /auth/verify` — JWT validieren
  - `POST /auth/set-password` — Erstes Passwort setzen via Magic Link Token
  - `POST /auth/forgot-password` — neuen reset_token generieren + Ionos SMTP verschicken
- [ ] Ionos SMTP einbinden — Magic Link / Passwort-Reset Emails versenden
- [ ] `login.html` — Submit-Flow verdrahten (POST /auth/login, Weiterleitung nach Rolle)
- [ ] `set-password.html` — Seite für Erstzugang / Passwort-Reset via Token

### Customer
- [x] `Interface/customer/index.html` — Customer Dashboard ✓ (v0.3.0)
- [x] `customer/index.html` — Background-Animationen für alle Sektionen ✓ (v0.4.0)
- [x] `customer/index.html` — Mouse-Parallax für alle 6 Sektions-Animationen ✓ (v0.5.0)
- [x] `customer/index.html` — Scroll-proportionales Fade-In/Out für alle Animations-Canvases ✓ (v0.5.0)
- [ ] `customer/index.html` — Auth-Check einbauen (JWT via GET /auth/verify beim Laden)
- [ ] `customer/index.html` — Nutzername dynamisch aus JWT laden (Begrüßung personalisieren)
- [ ] `customer/index.html` — Verträge dynamisch aus Airtable/Server laden (statt Placeholder)
- [ ] `customer/index.html` — Reclaim.ai iframe für Terminbuchung einbauen

### Employee
- [ ] `Interface/employee/index.html` — Employee Dashboard nach Login (Time-Tracker, Arbeitszeit, Pausen)
