# CLAUDE.md – Satz (Calisthenics-Log-PWA)

## Was diese App ist

"Satz" ist eine minimalistische PWA zum Dokumentieren von Bodyweight-/Calisthenics-Training.
Kernprinzip seit v10: **Checklisten-basierter Flow ohne laufende Timer/Stoppuhren.**
Satzwerte (Wdh oder Sekunden) werden manuell eingetragen und abgehakt; Session-Start/-Ende
werden nur als Timestamps mitgeschrieben. Zielgerät: Smartphone im (Keller-)Gym →
**Offline-first ist Pflicht.** Footer-Branding: "Satz · läuft von allein".

## Dateien

- `index.html` – die komplette App (HTML + CSS + JS inline, ~1.150 Zeilen). Einzige Quelle der Logik.
- `service-worker.js` – Offline-Cache. `CACHE`-Konstante aktuell `satz-v10`.
- `manifest.json` – PWA-Metadaten (Farben dort sind die Wahrheit: bg `#FBFBFA`, theme `#FFFFFF`).
- `icons/` – 4 PNGs (64/192/512/maskable), Stoppuhr-Glyphe, dunkler Grund `#2B2926`.
- `Trainingsplan-Vorlage.xlsx` – Excel-Vorlage für den Plan-Import (Format s.u.).
- `Anleitung-Satz.pdf` – Nutzer-Doku. **Stand veraltet (beschreibt noch Timer-Version v1) – muss bei Feature-Änderungen mit aktualisiert werden.**

## Architektur & Stack – aktueller Stand

Das ist die bewusst gewählte Basis. Sie ist nicht in Stein gemeißelt: **Wenn eine Aufgabe
zeigt, dass diese Architektur an Grenzen stößt, sprich es aktiv an und schlage eine
Alternative vor** – aber wechsle sie nicht stillschweigend im Vorbeigehen.

- **Single-File:** Alle Logik lebt in `index.html`. Kein Aufsplitten ohne Anlass.
- **Kein Build-Step. Kein Framework. Kein npm.** Vanilla JS, plain CSS mit Custom
  Properties als Design-Tokens in `:root`.
- **Externe Abhängigkeiten (aktuell genau zwei, beide werden vom SW zur Laufzeit gecacht):**
  1. Google Fonts: Schibsted Grotesk (UI) + JetBrains Mono (Zahlen, Klasse `.num`,
     tabular-nums).
  2. SheetJS 0.18.5 via cdnjs (Excel-Import/-Export).
  Neue Abhängigkeiten sind möglich, aber begründungspflichtig – und nichts einbauen,
  was den Erstlade-Pfad oder die Offline-Fähigkeit ungeprüft verschlechtert.
- **Sprache:** UI-Texte und Code-Kommentare auf Deutsch.

## Design-Linie

- Notion-inspirierter Light-Look: Ink `#37352F`, Akzent-Blau `#2383E2`, Karten mit
  1px-Linien (`--line #EAE9E5`), Radius 12px, Tags in Pastell (blau=Wdh, grün=Halten,
  grau=neutral).
- **Kein Dark Mode vorhanden** – wäre eine bewusste Produktentscheidung, gern vorschlagen,
  aber nicht ungefragt einbauen.
- Mobile-first, `max-width: 560px`, Safe-Area-Insets beachtet, `--nav-h` für die
  Bottom-Tab-Bar. Große Touch-Targets, `:active`-Feedback statt Hover.
- Haptik: `vib()`-Helper (navigator.vibrate) für Bestätigungen.

## Datenmodell & Persistenz

- **localStorage mit In-Memory-Fallback** (Wrapper mit `get`/`set`, try/catch).
- Namespace `satz.*` – Keys: `satz.active` (laufende Einheit), `satz.sessions` (Verlauf),
  `satz.lib` (Übungsbibliothek inkl. letzter Werte), `satz.plans` (Trainingspläne),
  `satz.tech` (Technikkarten), `satz.settings`.
- Übungstypen (`mode`): `reps` (Wdh) · `hold` (Halten, Sekunden) · `time` (Zeit/Aufwärmen).
  Helper: `isTimed()`, `modeLabel()`, `modeTag()`.
- Features am Datenmodell: Supersatz-Verkettung (Pause erst nach dem Block),
  optionales Gewicht (kg) pro Satz, geführter Plan-Modus (`guide` mit `queue`/`idx`),
  Seed-Technikkarten für ~12 Standardübungen.
- **JSON-Backup**: Export aller Stores als `satz-backup-YYYY-MM-DD.json` – bei
  Schema-Änderungen den Import kompatibel halten.
- Bei Schema-Änderungen: Migrationslogik für Bestandsdaten mitliefern, niemals
  bestehende `satz.*`-Daten stillschweigend invalidieren.

## Excel-Plan-Format (Import/Export)

Header (15 Spalten, exakt diese Reihenfolge):
`Plan | Übung | Typ | Ziel | Sätze | Pause | Gerät | Gewicht | Supersatz | Setup | Ablauf | Cues | Atmung | Fehler | Video`
- Typ-Parsing ist tolerant: `Wdh`/`Halten`/`Zeit` (auch engl. Präfixe hold/time).
- Mehrere Zeilen mit gleichem Plan-Namen = ein Plan in dieser Reihenfolge.
- Import ersetzt gleichnamige Pläne. `.csv` wird ebenfalls unterstützt (deutsches Excel).
- Format-Änderungen immer synchron in: Parser, Export, `Trainingsplan-Vorlage.xlsx`
  und Anleitung.

## Versionierung & Deploy

- `APP_VERSION` + `APP_BUILD` in `index.html` (aktuell `v10` · `07.07.2026`) – bei jeder
  nutzersichtbaren Änderung hochzählen, wird im App-Footer angezeigt.
- **Synchron dazu die `CACHE`-Konstante in `service-worker.js` erhöhen** (`satz-v10` → `satz-v11`),
  sonst sehen installierte PWAs die alte Version.
- SW-Strategie: cache-first für die App-Shell; Fonts + cdnjs werden zur Laufzeit mitgecacht.
- Hosting: **GitHub Pages** (Repo-Root, Branch main). Deploy = Dateien committen/pushen.
  Kein Firebase, kein deploy.js bei diesem Projekt.

## Arbeitsweise mit mir (Jan)

- Bei mehreren sinnvollen Wegen: **Optionen kurz aufzählen** (Trade-offs in 1–2 Sätzen),
  eine Empfehlung markieren. Bei trivialen Entscheidungen ohne echten Trade-off: einfach machen.
- **Feature-Ideen und Verbesserungsvorschläge sind willkommen** – vorschlagen ja,
  aber nicht ungefragt bauen. Minimalismus ist das Produkt.
- Antworten und Commits knapp und auf Deutsch.
- Screenshots von mir sind Bug-Reports: Fehler selbst identifizieren und fixen.
- Vor Abgabe: JS-Syntax validieren (`node --check` auf extrahiertes Script).
- Bekannter Stolperstein aus der Historie: bei Re-Renders keine Eingabe-States zerstören
  (siehe Composer-Kommentar in index.html) und keine Event-Listener durch `innerHTML +=` verlieren.

## Leitplanken – nur nach Rücksprache ändern

Diese Punkte sind bewusste Entscheidungen. Wenn du bei einer Aufgabe merkst, dass eine
davon dem besseren Ergebnis im Weg steht: **weise explizit darauf hin und schlage die
Anpassung vor** – aber setze sie erst nach meinem Ok um.

- Build-Tooling, Bundler, Framework oder npm-Dependencies einführen.
- Die App in mehrere JS/CSS-Dateien aufsplitten.
- Laufende Timer/Stoppuhr-Logik wieder einbauen (bewusst in v10 entfernt).
- Neue externe Requests, die Offline-Fähigkeit oder Erstlade-Pfad verändern.
- `satz.*`-Bestandsdaten ohne Migration invalidieren oder das Backup-Format brechen.
- Dark Mode, Accounts oder Cloud-Sync.
