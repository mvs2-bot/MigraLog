# Migräne Tracker (MigraLog)

Single-File-PWA aus der mvs2-App-Familie: Migränetage, Schmerzstärke, Auslöser und Medikamente
mit wenigen Fingertipps erfassen. Gleiche Designsprache und gleicher Kontozugang wie DailyMind.

## Dateien

| Datei | Inhalt |
| --- | --- |
| `index.html` | die komplette App: HTML, eigenes CSS, Alpine.js und supabase-js über CDN |
| `manifest.json`, `sw.js` | installierbar, App-Hülle offline verfügbar |
| `icon.svg`, `icon.png`, `apple-touch-icon.png`, `icons/` | App-Icon im Langschatten-Stil (Kopf mit Blitz auf Pflaume), maskable und Favicon |
| `shared/ios-nav.js` | Kopie aus `mvs2-bot.github.io` (Zurück/Neu laden in der iOS-Web-App) |
| Datenbank | Migration `…_migralog_core` im Repo `supabase-core` |

## Erfassung (ein Eintrag pro Tag)

Schmerzstärke 0–10, Phase (Warnzeichen/Aura, aktiver Anfall, abklingend, schmerzfrei), Begleitsymptome,
mögliche Auslöser (auch eigene), Medikation (auch eigene) mit Einnahmezeit und Wirkung, Notiz.
Unter „Mehr Details“: Ort des Schmerzes, Beginn, Einschränkung im Alltag.
Gespeichert wird automatisch. Ohne Netz bleibt der Eintrag auf dem Gerät und wird nachgeschickt.

## Auswertung

- Kalender mit Farbe je Tag: Salbei schmerzfrei, Bernstein 1–3, Orange 4–6, Himbeere 7–10.
- Migränetage (Monat und Vormonat), Ø Schmerzstärke, häufigster Auslöser (90 Tage) und stärkster Tag,
  schmerzfreie Tage in Folge (Tage ohne Eintrag zählen als schmerzfrei).
- Ab 10 Tagen mit Schmerzmitteln im Monat ein sanfter Hinweis (`MED_DAYS_HINT`).
- Export als CSV für den Arzttermin (Profilmenü oder „Verlauf“).

## Rollen und Datenschutz

- Zugang mit mindestens Rolle **Mitglied** für die App `migralog` (Admin-Einstellungen im Dashboard).
- Einträge sind **strikt privat** (RLS): auch Admins sehen keine fremden Einträge. Admins bekommen
  im Profilmenü den Link zu den Admin-Einstellungen.
- Name, Profilbild und Hell/Dunkel kommen aus dem gemeinsamen Konto (`overall_profiles`, `overall-theme`).

## Design

Wie DailyMind: Systemschrift, Überschriften in New York, weiße Karten auf `#f2f2f7`, dunkel `#1c1c1e` auf Schwarz.
Akzent Pflaume `#7a4a8f` (dunkel `#cfa6e6`). Handy bis 1023 px einspaltig, ab 1024 px eigenes Layout
(Erfassung links, Kalender und letzte Einträge rechts, vier Statistik-Karten darunter, max. 1200 px).
Bei Änderungen die Cache-Version in `sw.js` hochzählen.
