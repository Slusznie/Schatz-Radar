# Schatz-Radar – kostenlose Tablet-App (PWA)

Ersatz für die FlutterFlow-App: Die App zeigt die Funde aus der Schatzsuche live auf dem Android-Tablet und hat einen Max-Preis-Rechner eingebaut. Sie besteht aus einer einzigen HTML-Datei und braucht keinen eigenen Server und kein Konto. Die Daten kommen direkt aus dem ntfy-Thema.

## Dateien
| Datei | Zweck |
|---|---|
| `index.html` | Die komplette App (CSS und JS inline, keine Abhängigkeiten) |
| `manifest.webmanifest` | Für „Zum Startbildschirm hinzufügen“ / App installieren |
| `icon.svg`, `icon-192.png`, `icon-512.png` | App-Icon |
| `sw.js` | Service Worker (cacht nur die App-Hülle, nie ntfy-Daten; läuft nur über https) |
| `tests/test-parse.js` | 22 Testgruppen für Parser, Zusammenführen, Countdown, Rechner, Service Worker und Manifest: `node tests/test-parse.js` |
| `screenshot*.png` | Screenshots mit **Demo-Daten** (nicht echt) |

## Einrichtung
1. App öffnen. Beim ersten Start erscheint **⚙️ Einstellungen**.
2. Das ntfy-Thema eintragen (oder den ganzen Link `https://ntfy.sh/…` einfügen) und auf **Speichern & verbinden** tippen.
   Das Thema wird nur im Speicher des Tablets (localStorage) abgelegt und steht **nicht** in der App-Datei. Deshalb darf die Datei auch öffentlich gehostet werden.
3. Mit **Verbindung testen** siehst du, wie viele Nachrichten da sind und wie viele davon echte Funde sind.
4. Für Dauerbetrieb oben auf **⚡** tippen, damit der Bildschirm wach bleibt (das geht nur über https).

`index.html?demo=1` zeigt Beispiel-Daten mit dem Abzeichen „DEMO-DATEN“. Das ist nur für Vorschau und Screenshots gedacht.

## Was die App macht
- **Kopf:** Uhrzeit (Europe/Berlin), Abzeichen „LIVE DÖRPEN“ (rot = offline), ⚡ Wach halten und 🌙/☀️ Hell/Dunkel.
- **Gewinn heute:** Summe „Gewinn netto“ aller grünen Funde von heute. Gelbe Funde werden daneben extra angezeigt.
- **Countdown** zur nächsten Suchrunde: 08:13, 13:13, 18:13 und 22:13 Uhr.
- **Kacheln und Tabs:** Grün (Sofort), Gelb (Prüfen) und Endet bald (Ende in weniger als 3 Std.). Ab 900 px Breite (Tablet quer) erscheinen drei Spalten nebeneinander.
- **Neue Funde** fahren animiert ein und leuchten auf. Danach glühen sie noch 10 Min leicht. Ein **Mega-Schatz** (Gewinn > 100 €, einstellbar, oder „MEGA“ im Titel) wird 7 Sek. im Vollbild gezeigt, das Tablet vibriert.
- **Nachtmodus** 23–6 Uhr: Die App wird gedimmt, nur das Mega-Vollbild leuchtet weiter.
- **Daten:** Die App fragt alle 60 Sek. `GET <server>/<thema>/json?poll=1&since=24h` ab und hört zusätzlich live auf `<server>/<thema>/sse` (abschaltbar). Hinweis: ntfy.sh speichert Nachrichten nur ca. 12 Std., ältere Funde fallen also heraus.
- **🧮 Rechner:** Mit dem Knopf auf einer Fund-Karte übernimmst du den Fund in den Rechner.

## Max-Preis-Rechner (Formel)
```
erwartet  = Verkaufswert repariert                          (Defekt klar)
          = 0,6 × Verkaufswert + 0,4 × Teilewert            (Defekt unklar)
          (nie unter dem Teilewert)
Erlös     = erwartet × (1 − Gebühren 13 %) − Versand raus
direkt    = Versand rein + Ersatzteile
Lohn      = Stunden × Stundensatz (Std. 15 €)
Max-EK    = Erlös − direkt − (Wunschgewinn 40 € + Lohn) / (1 − Kollegenanteil 25 %)

Rohgewinn = Erlös − direkt − EK          (EK = Angebotspreis oder Max-EK)
Dein Gew. = Rohgewinn × (1 − Kollegenanteil)
€/Std.    = Dein Gewinn / Stunden  →  Ampel: grün ≥ 20, gelb 10–20, rot < 10 (oder Max-EK ≤ 0)
Bei 0 Stunden: €/Std. = „–“, Ampel nur nach Gewinn: grün ≥ Wunschgewinn, gelb > 0, rot ≤ 0
```
Beim Max-EK bleiben dir nach Kollegenanteil also genau der Wunschgewinn plus dein Stundenlohn. Alle Standardwerte lassen sich ändern und werden gespeichert.

## Nachrichtenformat (für Maier / die Such-Bots)

### A) Text-Format (wie bisher)
```
Titel:  💎 SCHATZ GEFUNDEN – <Gerät>       → grün
        🔥💎 MEGA-SCHATZ – <Gerät>          → Mega (Vollbild)
        🟡 Mal anschauen – <Gerät>          → gelb
        ⏰ Endet in 30 Min – <Gerät>         → Erinnerung: aktualisiert den bestehenden Fund (siehe unten)
Text (Markdown erlaubt, Emojis davor egal, eine Angabe pro Zeile):
        🟢 **Ampel:** Grün | Gelb | Rot     (Rot wird nicht angezeigt; ohne Zeile zählen die Tags
                                             red_circle / yellow_circle / green_circle)
        💶 **Preis:** 22 € inkl. Versand    (oder „… Abholung Dörpen“ → Chip „Dörpen“)
        📈 **Gewinn netto:** ca. 45 €
        🔧 **Defekt:** ohne RAM/SSD
        📍 **Quelle:** Kleinanzeigen · Versand   (erstes Wort vor „·“ = Plattform-Chip)
        ⏳ **Endet:** 18:30 | in 2 Std 15 Min | 2026-11-18T18:30:00+01:00 | 1763486400 (Unix-Sek. oder -ms)   (optional)
Header: Click: <Anzeigen-Link>   → Knopf „Öffnen“
        Attach: <Bild-URL>       → Vorschaubild (attachment.url)
```
Zahlen dürfen deutsch geschrieben sein („1.234,50 €“, „ca. 45 €“), auch negativ („-12 €“ → angezeigt als „−12 €“).
Feldnamen müssen **genau** stimmen: `Preis:` (oder `Preis inkl. Versand:`, `EK:`, `Einkaufspreis:`) und `Gewinn netto:` (oder `Gewinn:`). `Preis neu:` oder `Gewinn brutto:` werden ignoriert. Kommt ein Feld doppelt vor, gilt die erste Zeile.
Vorschaubilder werden nur über **https** geladen.

### Ein Fund pro Anzeige: Erinnerungen und Doppelungen
- Die App fasst Nachrichten zum selben Fund zusammen. Erkannt wird das am **Link** (`Click` bzw. `link`, ohne `#` und `/` am Ende). Fehlt der Link, zählen **Gerät + Quelle**.
- Eine **Erinnerung** (Titel beginnt mit ⏰ oder „Erinnerung“) aktualisiert nur die Endzeit des bestehenden Fundes, und der Fund erscheint dann unter „Endet bald“. Sie zählt **nicht** noch einmal in „Gewinn heute“, und es gibt kein zweites Mega-Vollbild.
- Eine Erinnerung ohne Ampel und ohne bekannten Fund ist **nicht** automatisch grün. Sie steht nur unter „Endet bald“ (orange).
- Für die Bots heißt das: Bei Erinnerungen bitte denselben `Click`-Link wie beim Original mitschicken.

### B) Optionaler DATA-Block (hat Vorrang vor dem Text)
Eine Zeile im Nachrichtentext, die mit `DATA:` beginnt, gefolgt von JSON:
```
DATA: {"geraet":"HP EliteDesk 800 G4 Mini","preis":30,"gewinn":52,"defekt":"Netzteil fehlt","quelle":"Kleinanzeigen · Abholung Dörpen","link":"https://…","ampel":"gruen","ende":"2026-11-18T18:30:00+01:00"}
```
| Feld | Typ | Bedeutung |
|---|---|---|
| `geraet` | Text | Gerät oder Teil |
| `preis` | Zahl oder Text | Einkaufspreis inkl. Versand in € |
| `gewinn` | Zahl oder Text | Gewinn netto in € |
| `defekt` | Text | Defekt oder Zustand |
| `quelle` | Text | Plattform, z. B. „eBay · Versand“ |
| `link` | URL | Anzeige (nur http/https) |
| `ampel` | `gruen` / `gelb` / `rot` | Status (`rot` wird ausgeblendet) |
| `ende` | ISO-Zeit, Unix-Sekunden/-Millisekunden (Zahl oder Text), „18:30“ oder „in 40 Min“ | Auktionsende (optional) |
| `mega` | true/false | Mega-Schatz erzwingen (optional) |
| `bild` | https-URL | Vorschaubild (optional, sonst `Attach`) |

Fehlende Felder im DATA-Block werden aus dem Text bzw. Titel ergänzt. Der Block kann auch am Ende einer normalen, schön formatierten Nachricht stehen. In der ntfy-App sieht man ihn dann als letzte Zeile.

Beispiel:
```
curl -H "Title: 💎 SCHATZ GEFUNDEN – Lenovo M720q" -H "Tags: moneybag,green_circle" -H "Priority: high" \
     -H "Markdown: yes" -H "Click: https://www.kleinanzeigen.de/s-anzeige/…" \
     -d $'🟢 **Ampel:** Grün\n💶 **Preis:** 22 € inkl. Versand\n📈 **Gewinn netto:** ca. 45 €\n🔧 **Defekt:** ohne RAM/SSD\n📍 **Quelle:** Kleinanzeigen · Versand' \
     https://ntfy.sh/<thema>
```

### Werden ausgeblendet
- Titel, in denen eines dieser Wörter als **ganzes Wort** steht: „Test“, „Check“, „ntfy-Check“, „DESIGN-MUSTER“, „Schatz-Test“, „Team-Test“, „Testfund“ oder „Testnachricht“.
  „getestet“, „Kabeltester“, „Checkpoint“ oder „Testo“ sind dagegen echte Funde und werden angezeigt.
- Text mit „kein echter Fund“, „DESIGN-MUSTER“, „Testfund“ oder „Testnachricht“
- Nachrichten, die nicht nach Fund aussehen (kein 💎/🟡/⏰/Schatz im Titel, kein DATA und nicht Preis + Gewinn)
- Funde mit Ampel Rot

## Technik / Sicherheit
- ntfy.sh erlaubt CORS (`Access-Control-Allow-Origin: *`, geprüft am 18.11.2026 für `/json` und `/sse`).
- Alle Texte aus Nachrichten werden escaped. Für Links sind nur http(s) erlaubt, für Bilder nur https. Die Bild-URL wird kodiert und per `element.style` gesetzt.
- Der Service Worker cached nur erfolgreiche Antworten (`response.ok`). Das Manifest-Icon ist nur „any“, nicht „maskable“.
- Wer das Thema kennt, kann mitlesen und Fake-Funde schicken. Das Thema darf deshalb nicht weitergegeben werden.
