# CLAUDE.md – MigraLog (KopfschmerzTracker)

Single-File-PWA der mvs2-App-Familie unter `/MigraLog/` auf GitHub Pages. Kopfschmerztage, Schmerzstärke,
Auslöser und Medikamente erfassen und auswerten. Designsprache und Konto wie DailyMind / Dashboard.
Kein Build, kein npm: reines HTML/CSS/JS, Alpine.js + supabase-js per CDN. Sprache in UI, Kommentaren, Commits: Deutsch.

## Arbeitsweise (Token-Effizienz)
- Nicht das ganze Repo einlesen. `index.html` (~1500 Zeilen) per `grep -n` anspringen (Karte unten).
- Vor dem PR selbst prüfen: JS-Syntax, Alpine-Bindings ↔ Methoden/Konstanten, IDs, Design-Tokens, Dark Mode, Handy + Desktop.
- Bei Änderung an Architektur, Dateistruktur oder Schema diese Datei im selben PR anpassen (max. 150 Zeilen).
- Keine Secrets. Nur der öffentliche Publishable Key (`sb_publishable_…`) ist erlaubt; niemals `service_role`/secret Key.
- Bei jeder Änderung an gecachten Dateien Cache-Version in `sw.js` hochzählen (`V=P+"vNN"`, aktuell `v9`).

## Dateien
| Datei | Inhalt |
| --- | --- |
| `index.html` | komplette App: `<style>`, Konstanten, Markup, Alpine-Komponente `migraLog()` |
| `sw.js` | Service Worker: eigene Dateien Netz zuerst (4 s), CDN aus Cache, **Supabase nie cachen** |
| `manifest.json`, `icon.svg`, `icon.png`, `apple-touch-icon.png`, `icons/` | PWA, Icons |
| `shared/ios-nav.js` | **Kopie** aus `mvs2-bot.github.io/shared/` – nur dort ändern, dann hierher kopieren |
| `README.md` | Funktionsbeschreibung (Erfassung, Auswertung, Rollen, Design) |

Neue gecachte Datei → in `CORE` von `sw.js` aufnehmen und Version erhöhen.

## Karte `index.html` (ungefähre Zeilen, per grep verifizieren)
| Bereich | Suchbegriff |
| --- | --- |
| Theme vor Render (`html.dark` aus `localStorage['overall-theme']`) | `overall-theme` |
| Design-Tokens `:root` / `.dark` | `:root {`, `.dark {` |
| Breakpoints: Handy < 768 / ab 480 / Desktop ab 1024 px | `@media` |
| Konstanten: `SUPABASE_URL`, `APP_ID='migralog'`, `ACCOUNT_URL`, `ADMIN_URL` | `const APP_ID` |
| Listen: `TYPES`, `PHASES`, `SYMPTOMS`, `TRIGGERS`, `LOCATIONS`, `MEDS`, `EFFECTS`, `IMPACTS`, `PAIN_WORDS`, `PICKERS`, `RANGES`, `MED_DAYS_HINT` | `const PICKERS` |
| Linien-Symbole `IC` (SVG-Pfade, 24er viewBox) | `const IC` |
| Views: `loading`, `auth`, `noaccess`, App | `view ===` |
| Tabs: `today` (Erfassung), `hist` (Verlauf), `profile` | `tab ===` |
| Seitenfeld Eintrag (Desktop), Tab-Leiste (Handy) | `Seitenfeld`, `Tab-Leiste` |
| Helfer `isOffline`, `withTimeout` | `const withTimeout` |
| Komponente: `init`, `bootWatch`, `loadMe`, `checkAccess`, `submitAuth`, `signOut` | `async init` |
| Daten: `loadEntries`, `save`, `send`, `flushQueue`, `deleteEntry`, `goTo` | `async save` |
| Export: `exportCsv` (alle Einträge), `exportPdf` (Zeitraum aus Verlauf, Druckansicht `#printSheet` + `window.print`, S/W-Balken 0–5, CSS `@media print`) | `exportPdf` |
| Start-Notbremse (8 s „Neu laden“) | `Neu laden` |

Konstanten, die das Markup braucht, werden in der Komponente durchgereicht (`ACCOUNT_URL, ADMIN_URL, IC, …` nahe `migraLog()`).
Neue Konstante im Markup → dort ergänzen.

## Supabase-Anbindung
- Gemeinsames Core-Projekt, `supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY)`, supabase-js `2.45.4`, Alpine `3.14.1` (jsDelivr).
- **Schema liegt ausschließlich im Repo `supabase-core`** (Migrationen `…_migralog_core`, `…_migralog_headache_type`).
  Schemaänderung = dort neue Migration mit Test, eigener PR.
- Genutzt:
  - RPC `migralog_is_member()` (Zugangsprüfung, Ergebnis gecacht), `overall_app_role(p_app)` (Admin-Link).
  - Tabelle `overall_profiles` (`display_name`, `avatar_path`), Storage-Bucket `avatars` (Signed URL).
  - Tabelle `migralog_entries`: select (max. 1500), `upsert` mit `onConflict: 'user_id,entry_date'`, delete.
- `migralog_entries`: ein Eintrag pro Nutzer und Tag. Spalten: `entry_date`, `pain` (DB 0–10, UI 0–5),
  `headache_type` (migraine|tension|stabbing|dull), `phase` (aura|attack|fading|free), `started_at`,
  `locations[]`, `symptoms[]`, `triggers[]`, `meds[]`, `med_time`, `med_effect` (0–2), `impact` (0–2), `note`.
- RLS: Einträge strikt privat (nur `auth.uid()`, auch Admins nicht). Zugang ab Rolle `member` der App `migralog`.
- Login nur mit vorhandenem Konto (Admin legt an). Konto/Admin-Seiten liegen im Dashboard (`/konto.html`, `/admin.html`).

## Offline & lokale Daten
- Ungesendete Einträge in einer Warteschlange in `localStorage` (`migralog-queue-<uid>`), `flushQueue()` schickt nach.
- Weitere Keys: `migralog-me-<uid>` (Profil-Cache), `migralog-access-<uid>` (Zugangs-Cache), `overall-theme` (app-übergreifend).
- Netzaufrufe mit `withTimeout`; Offline-Fehler über `isOffline` erkennen, nie endlos laden.

## Design-Tokens (`:root`, Dark über Klasse `.dark` auf `<html>`)
- Flächen: `--bg #f2f2f7`, `--card #fff`, `--well`, `--hover`, `--line`, `--sep`; dunkel `--bg #000`, `--card #1c1c1e`.
- Text: `--ink #1c1c1e`, `--text #3a3a3c`, `--mute #6c6c70`.
- Akzent Pflaume: `--p1 #7a4a8f` (dunkel `#cfa6e6`), `--p1-soft`, `--on-p1`.
- Intensität: `--lv0` Salbei (schmerzfrei), `--lv1` Bernstein (1–2), `--lv2` Orange (3), `--lv3` Himbeere (4–5), je mit `-soft`.
- Status: `--ok`, `--warn`, `--danger`, `--danger-soft`.
- `--radius 18px`, `--radius-s 12px`, `--shadow`, `--pop`, `--font` (Systemschrift/SF), `--serif` (New York, Überschriften).
- Aliasse für `ios-nav.js`: `--border`, `--accent`, `--card-hover`, `--muted`.
- Apple-Designsprache: weiße Karten auf Grau, iOS-Segmente, Linien-Symbole statt Emojis, `prefers-reduced-motion/-transparency` beachten.
- Layout: Handy einspaltig mit schwebender Tab-Leiste; ab 1024 px „Heute“ zentriert 720 px, „Verlauf“ Raster max. 1200 px.

## Konventionen
- Neue Farben als Variable in `:root` **und** `.dark`, nicht hart im Markup.
- Neue Auswahl-Optionen nur in den Listen-Konstanten ergänzen; gespeichert werden die Klartexte bzw. `id`s.
- Bestehende DB-Werte bleiben gültig (z. B. alte `pain` > 5 zählen als stark).
- Gesundheitsdaten: nichts loggen, nichts an Dritte, Supabase-Antworten nicht im SW cachen.
