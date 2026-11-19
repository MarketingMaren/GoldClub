# GoldClub Level 4 – Weihnachtsfeier-App

Ein Escape Game vom Marketing-Team: Gäste scannen den QR-Code auf dem Poster, geben die PIN von der GoldClub-Karte und ihren Namen ein (ab dann läuft die Uhr) und müssen das Abo „GoldClub Level 4" so schnell wie möglich wieder kündigen. Die drei schnellsten Kündigungen bis 24 Uhr gewinnen.

Live unter **https://das-goldene-zeitalter-des-goldclubs.vercel.app/** (direkt unter der Startadresse). Die Lösungen liegen bewusst nicht in diesem Repo.

## Ablauf und Zeitplan
| Zeitraum (Berlin) | Was Gäste sehen |
|---|---|
| vor 27.11.2026, 18:30 | Sperrseite mit Tür und Countdown |
| 27.11.2026, 18:30 bis 23:59:59 | Spiel läuft |
| ab 28.11.2026, 00:00 | Nur noch die finale Rangliste mit den Gewinner:innen |

Preise: 1. Platz 100 € Hotel-Guthaben · 2. Platz Süße GoldClub-Karte · 3. Platz Goldenes Schreibtisch-Upgrade (in `app/index.html` unter `PRIZES`).

## Team-Menü (versteckt, jederzeit erreichbar)
Drei Wege, in jeder Phase (Sperrseite, Spiel, Spielende):
- Adresse **`/admin`** aufrufen, z. B. `https://das-goldene-zeitalter-des-goldclubs.vercel.app/admin`
- **5× schnell auf das „G“ der Tür** oder auf die Überschrift der Sperrseite tippen
- im Spiel und auf der Endseite **5× schnell auf das Logo** „GoldClub Level 4" oben tippen

Dann den Admin-Key eingeben (Umgebungsvariable `ADMIN_KEY` in Vercel). Er bleibt auf dem Gerät gespeichert, bis man im Menü „Abmelden" wählt. Nach 20 falschen Versuchen ist der Zugang für 10 Minuten gesperrt.

- **Vorschau nur auf diesem Gerät:** „Spiel läuft", „Spiel vorbei", „Sperrseite", „Vorschau beenden". Zum Testen vor dem Start. Testläufe sind als „Test" markiert und erscheinen nicht in der echten Rangliste und nicht in den Live-Meldungen echter Gäste. Ein roter Hinweis unten links zeigt, dass die Vorschau aktiv ist.
- **Für alle:** „Jetzt freischalten", „Jetzt beenden", „Wieder sperren", „Zeitplan (automatisch)". Wirkt sofort für alle Gäste.
- **Monitoring:** `/monitor` (fragt nach dem Admin-Key). Zeigt alle Teilnehmenden mit Status (drin / gekündigt), aktueller Station, Bestzeit, Läufen, Glücksrad-Drehs, Sackgassen und Gewinn, dazu eine Auswertung, wer welche Station erreicht hat. Dort lässt sich auch die Rangliste leeren.

## Ordner
| Datei/Ordner | Inhalt |
|---|---|
| `app/index.html` | Die komplette Oberfläche, alle Texte zum Anpassen |
| `app/hunde/` | Optional: eigene Fotos hässlicher Hunde für die Fehlerseiten (sonst gezeichnete Hunde) |
| `app/captcha/` | Optional: Fotos + `captcha.json` für das Foto-Captcha |
| `lib/core.js` | Server-Logik (PIN, Zeitfenster, Zeitmessung, Rangliste, Live-Meldungen, Monitoring) |
| `lib/assets.js` | Automatisch erzeugt (`npm run embed`), nur für Deploy ohne GitHub |
| `server.js` | Start auf dem eigenen Rechner |
| `api/index.js` + `vercel.json` | Start auf Vercel |

## Was nicht ins Repo gehört
Per `.gitignore` ausgeschlossen: `zugangsdaten.txt`, `.env*`, `.vercel/`, `data.json`. Beim Hochladen über die GitHub-Weboberfläche greift die `.gitignore` nicht, dann diese Dateien nicht mit hineinziehen.

## Schutz
- **Adresse:** ohne `TICKET_TOKEN` läuft das Spiel unter der Startadresse; mit `TICKET_TOKEN` nur unter `/<TICKET_TOKEN>`. Repo-Dateien werden nie direkt ausgeliefert (dafür sorgt `vercel.json`).
- **Zeitfenster** und **PIN** prüft der Server, nicht nur die Oberfläche. Max. 10 falsche PINs pro 10 Minuten.
- Zeiten misst der Server, die Warteschleife lässt sich nicht abkürzen.

## Einstellungen (Umgebungsvariablen in Vercel)
| Variable | Bedeutung | Standard |
|---|---|---|
| `TICKET_TOKEN` | Optional: geheimer Teil der URL. Leer = Startadresse | leer |
| `EVENT_PIN` | PIN auf der GoldClub-Karte | `6767` |
| `ADMIN_KEY` | Schlüssel für Team-Menü, Monitoring und Reset (leer = aus). Nie in den Code schreiben | leer |
| `GAME_START` | Spielbeginn | `2026-11-27T18:30:00+01:00` |
| `GAME_END` | Spielende | `2026-11-27T23:59:59+01:00` |
| `QUEUE_SECONDS` | Länge der Warteschleife (mind. 25) | `60` |
| `SESSION_SECRET` | Optional, sonst aus Token und PIN abgeleitet | – |

Änderungen an Variablen greifen erst mit dem nächsten Deploy (Commit oder „Redeploy").

## Lokal testen
```bash
ADMIN_KEY=test node server.js
```
Dann http://localhost:3000 öffnen. Vor dem 27.11. erscheint die Sperrseite: 5× aufs „G“ tippen, Admin-Key `test`, „Spiel läuft". PIN `6767`. Daten liegen in `data.json`.

## Online mit Vercel (über GitHub)
Jeder Commit auf `main` wird automatisch veröffentlicht. Einmalig im Vercel-Projekt: Umgebungsvariablen setzen, unter **Storage** „Upstash for Redis" (Free, Frankfurt) verbinden.

Wechsel auf das GoldClub-Repo später: im Vercel-Projekt unter **Settings → Git** das Repo umstellen. Adresse, Einstellungen und Rangliste bleiben erhalten.

Hinweis: Der Hobby-Tarif von Vercel ist laut Vercel für nicht-kommerzielle Nutzung gedacht; für eine interne Feier ggf. kurz klären.

## Rangliste leeren
Im Monitoring unten „Rangliste leeren" (zweimal klicken). Alternativ:
```bash
curl -X POST -H "x-admin-key: <ADMIN_KEY>" https://das-goldene-zeitalter-des-goldclubs.vercel.app/api/admin/reset
```
Nach der Feier zusätzlich die Upstash-Datenbank im Vercel-Dashboard löschen. Gespeichert sind nur Spitznamen, Zeiten und erreichte Stationen.

## Vor Ort
- **Poster:** QR-Code auf https://das-goldene-zeitalter-des-goldclubs.vercel.app/
- **GoldClub-Karte:** PIN aufdrucken (muss `EVENT_PIN` entsprechen).
