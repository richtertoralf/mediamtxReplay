# mediamtxReplay

## Motivation
Bei Sportproduktionen oder Livestreams kommt es immer wieder vor, dass ein spannender Moment sofort noch einmal gezeigt werden soll – ein klassisches **Replay**.  

Bisher ist das mit **OBS** oder **vMix** nur umständlich möglich:  
- Entweder muss man dauerhafte Aufzeichnungen anlegen und diese manuell suchen,  
- oder externe Replay-Systeme einsetzen, die teuer und komplex sind.  

Mit **MediaMTX** als zentralem Stream-Router entsteht aber eine neue Möglichkeit:  
Jeder veröffentlichte Stream kann automatisch aufgezeichnet werden.  
Der integrierte **Playback-Server** erlaubt den gezielten Abruf einzelner Zeitspannen.  

Damit ist die technische Grundlage für ein leichtgewichtiges Replay-System gegeben, das vollständig auf **Open-Source-Komponenten** basiert.

Das Ziel: **Einfaches, schnelles Replay aus OBS heraus**, steuerbar über ein **StreamDeck** oder Hotkeys, ohne dass teure Zusatzhardware nötig ist.

---

## Ziele des Projekts
- **Replay von allen auf einem MediaMTX-Server publizierten Streams**  
  Jeder eingehende Stream wird automatisch für eine kurze Zeit (Ringpuffer) aufgezeichnet.

- **Abruf von festen Replay-Dauern**  
  Über vordefinierte Tasten oder Buttons sollen direkt **10 s, 20 s oder 40 s** zurückliegende Sequenzen abrufbar sein.

- **StreamDeck-Integration**  
  - Erster Klick: Stream auswählen (z. B. „Kamera 1“ oder „Cam Clock“).  
  - Zweiter Klick: gewünschter Rücksprung (10 s, 20 s, 40 s).  
  - Automatische Generierung der passenden `/get`-URL vom MediaMTX-Server.  

- **Kompatibilität mit OBS**  
  Die erzeugte URL soll in OBS abspielbar sein, z. B. über eine Media Source oder eine Browser Source.  
  Ziel ist eine **praktikable Bedienung** im Live-Betrieb: möglichst wenige Klicks, sofortiges Replay.

- **Minimalinfrastruktur**  
  - Keine zusätzlichen Video-Server.  
  - Alles läuft auf vorhandener MediaMTX-Instanz plus einem kleinen Replay-Helper-Dienst (ähnlich wie beim MediamtxMonitor).  

---

## Praktikabilität für OBS
Da OBS keine eingebaute Replay-Logik für externe Streams hat, gibt es zwei mögliche Ansätze:

### 1. Media Source Ansatz (direkte HTTP-URL)
- Replay-Helper generiert die passende `/get`-URL.  
- OBS-Media Source (mit „Lokale Datei“ deaktiviert) spielt die URL ab.  
- Vorteil: Keine Extra-Software im Signalweg, sehr direkt.  
- Nachteil: Jede URL ist „einmalig“ → Quelle in OBS muss neu gesetzt oder neu geladen werden.

### 2. Browser Source Ansatz (HTML-Wrapper)
- Replay-Helper stellt eine kleine HTML-Seite bereit (z. B. `/replay/latest?path=cam1&duration=20`).  
- In dieser Seite ist ein `<video>`-Tag, das den Replay-Clip automatisch lädt.  
- OBS nutzt nur eine feste Browser Source („Replay“).  
- Vorteil: OBS muss nicht angepasst werden, die Replay-Steuerung läuft nur über die URL-Parameter.  
- Sehr einfach mit StreamDeck-Buttons kombinierbar.  

👉 Für den Live-Einsatz ist Variante **2 (Browser Source)** meist praktikabler, weil man in OBS nur **eine** Replay-Quelle anlegt und sie per StreamDeck mit neuen Clips „füttern“ kann.

---

## Infos

- [📄 settings.md](config/settings.md) – Settings in mediamtx.yml

--- 

## Nächste Schritte
- [ ] Python-Service (FastAPI/Flask) entwickeln, der für jeden Pfad die „letzten X Sekunden“ berechnet und als `/get`-URL ausliefert.  
- [ ] Einbindung in StreamDeck (HTTP Request Plugin).  
- [ ] Dokumentation mit Beispielen für OBS (Media Source vs. Browser Source).  
- [ ] Tests im Live-Betrieb mit MediaMTX-Teststreams.  


---

neu gedacht:

---

# mediamtxReplay

**Local replay workstation for MediaMTX streams with N3 control and OBS operator output**

## Ziel

`mediamtxReplay` ist eine einfache, robuste Replay-Architektur für Live-Produktionen mit MediaMTX.

Die Grundidee ist bewusst pragmatisch:

- der zentrale **MediaMTX-Ingest-Server** bleibt stabil und geschützt
- das eigentliche **Replay** passiert **lokal auf dem Operator-PC**
- **OBS Studio** ist nur der **Arbeitsplatz des Replay-Operators**
- ein **N3 Streamdeck** steuert Auswahl, Spulen und Start des Replays
- ein kleiner **Replay-Controller** verbindet N3, lokales MediaMTX und OBS

Das Projekt richtet sich an kleine und mittlere Produktionen, bei denen ein eigener, verständlicher Replay-Arbeitsplatz wichtiger ist als komplexe Broadcast-Infrastruktur.

---

## Grundprinzip

Die Live-Streams laufen zunächst ganz normal in einen zentralen MediaMTX-Server.

Auf dem **OBS-/Operator-PC** läuft zusätzlich ein **zweites lokales MediaMTX**, das sich nur die für Replay nötigen Streams holt. Dadurch bleibt die Bedienung lokal, OBS liest lokal, und Netzwerkprobleme zwischen Operator-PC und zentralem Ingest stören nicht direkt die Vorschau in OBS.

Das Replay wird über die **Playback-Funktion des lokalen MediaMTX** bereitgestellt. Das **N3 Streamdeck** steuert diese Funktion über einen kleinen lokalen Dienst. OBS zeigt die Replay-Quellen an und gibt den gewählten Replay-Ausgang an den Bildmischer weiter.

---

## Zielarchitektur

```text
Encoder / Kameras
   ↓
[Edge MediaMTX]
   zentrale Live-Kreuzschiene
   - ingest
   - live distribution
   - stabil / geschützt
   ↓
[Operator-PC / OBS-PC]
   lokales MediaMTX
   - zieht nur replay-relevante Streams
   - hält lokale Playback-Basis vor
   ↓
   Replay-Controller
   - wählt Stream
   - verwaltet Cursor / MARK / LIVE / RESET
   - steuert Playback
   ↓
   OBS Studio
   - Vorschau für den Replay-Operator
   - Program-Out für Replay
   ↓
Bildmischer / Regie
```

---

## Rollen der Komponenten

## 1. Edge MediaMTX

Der zentrale MediaMTX-Server ist die eigentliche **Live-Kreuzschiene**.

### Aufgaben

- Streams von Encodern oder Kameras annehmen
- Live-Streams im Netz bereitstellen
- möglichst stabil und geschützt laufen

### Nicht seine Aufgabe

- keine Bedienlogik
- keine Replay-Steuerung
- kein OBS
- keine Operator-Oberfläche

Der Edge-Server soll im Idealfall nur das tun, was für den Live-Betrieb nötig ist.

---

## 2. Lokales MediaMTX auf dem Operator-PC

Das lokale MediaMTX ist die **Replay-Kreuzschiene**.

### Aufgaben

- nur die für Replay relevanten Streams vom Edge holen
- diese Streams lokal für OBS bereitstellen
- lokal die Recording-/Playback-Basis für Replay vorhalten

### Vorteil

OBS arbeitet nur noch mit lokalen Quellen. Dadurch wird der Operator-Arbeitsplatz ruhiger und robuster.

---

## 3. Replay-Controller

Der Replay-Controller ist ein kleiner lokaler Dienst, zum Beispiel in Python.

Er ist die eigentliche Logikschicht zwischen **N3**, **lokalem MediaMTX** und **OBS**.

### Aufgaben

- gewählten Replay-Stream merken
- Spulposition verwalten
- MARK setzen
- LIVE / RESET behandeln
- Playback gegen lokales MediaMTX steuern
- Status für N3 und OBS bereitstellen

### Wichtig

Der Controller ist bewusst klein. Keine große Plattform, keine Datenbankpflicht, keine unnötige Komplexität.

---

## 4. N3 Streamdeck

Das N3 ist die Bedienoberfläche für den Replay-Operator.

### Geplante Bedienlogik

- **6 Tasten:** Stream-Auswahl
- **3 Encoder:** Spulen in verschiedenen Schrittweiten
- **Encoder-Druck:** `STREAM_AB`
- **untere Tasten:** `LIVE`, `MARK`, `RESET`

Das N3 spricht nicht direkt mit MediaMTX, sondern mit dem lokalen Replay-Controller.

---

## 5. OBS Studio

OBS ist **nicht** die Replay-Engine.

OBS ist der **Arbeitsplatz des Replay-Operators**.

### Aufgaben

- lokale Replay-Quellen anzeigen
- den aktuell gewählten Replay-Stream sichtbar machen
- Replay sauber ausgeben
- Program-Out für die Regie liefern

### Nicht die Aufgabe von OBS

- nicht selbst zurückspulen
- nicht selbst die Replay-Logik verwalten
- nicht selbst MediaMTX ersetzen

OBS bleibt bewusst Bedien- und Ausgabeplatz.

---

## Bedienablauf

Ein typischer Ablauf sieht so aus:

1. Das lokale MediaMTX stellt mehrere Replay-Quellen lokal bereit.
2. OBS zeigt diese lokalen Quellen.
3. Der Operator wählt mit dem N3 einen Stream aus.
4. Der Replay-Controller setzt diesen Stream als aktiv.
5. Über die Encoder wird die gewünschte Stelle zurückgespult.
6. Mit `STREAM_AB` wird das Replay gestartet.
7. OBS gibt den Replay-Output aus.
8. Die Regie nimmt diesen OBS-Ausgang am Bildmischer ins Live-Programm.

---

## Warum diese Architektur?

Diese Trennung hat mehrere Vorteile:

### Stabilität

Der zentrale Ingest-Server bleibt frei von Bedienlogik und Replay-Experimenten.

### Lokale Bedienung

OBS und der Operator arbeiten lokal auf derselben Maschine.

### Weniger Netzstress in OBS

OBS zieht nicht ständig direkt an entfernten Quellen, sondern liest lokal.

### Klare Zuständigkeiten

- Edge MediaMTX = live
- lokales MediaMTX = replay
- Replay-Controller = Logik
- OBS = Vorschau und Ausgabe
- N3 = Bedienung

### Schrittweise entwickelbar

Das System kann klein anfangen und später erweitert werden.

---

## Projektstatus

Dieses Repository ist die Weiterentwicklung einer ersten Ideensammlung.

Die ursprüngliche Grundidee war:

- MediaMTX-Playback für Replay nutzen
- per Streamdeck einen Clip oder Startpunkt auswählen
- das Ergebnis in OBS sichtbar und nutzbar machen

Inzwischen ist die Zielarchitektur klarer:

- **nicht der zentrale Ingest-Server** stellt direkt den Operator-Replay-Arbeitsplatz bereit
- stattdessen läuft **lokal auf dem OBS-/N3-PC ein zweites MediaMTX**
- dieses lokale MediaMTX zieht nur replay-relevante Streams vom Hauptsystem
- das eigentliche Operator-Replay wird lokal gesteuert

Damit wird aus der ursprünglichen Idee ein vollständiger **Replay Operator Workstation**-Ansatz.

---

## Geplante Projektstruktur

```text
mediamtxReplay/
├─ README.md
├─ docs/
│  ├─ architecture.md
│  ├─ workflows.md
│  └─ obs-setup.md
├─ config/
│  ├─ mediamtx-edge.md
│  ├─ mediamtx-local.md
│  └─ settings.md
├─ controller/
│  ├─ replay-controller.py
│  └─ requirements.txt
├─ n3-plugin/
│  ├─ manifest.json
│  ├─ plugin.js
│  └─ images/
└─ examples/
   ├─ obs-scenes.md
   └─ api-examples.md
```

---

## Minimaler Entwicklungsplan

## Phase 1 – Architektur dokumentieren

- README aktualisieren
- Rollen der Hosts sauber beschreiben
- Bedienlogik festhalten

## Phase 2 – lokales MediaMTX testen

- ausgewählte Streams vom Edge holen
- lokale Bereitstellung für OBS prüfen
- Recording/Playback lokal testen

## Phase 3 – Replay-Controller bauen

- aktiven Stream verwalten
- Spul-Logik umsetzen
- einfache API lokal bereitstellen

## Phase 4 – N3 anbinden

- Plugin mit offiziellem SDK erstellen
- Tasten und Encoder an den Controller koppeln
- Statusrückmeldung auf dem N3 anzeigen

## Phase 5 – OBS-Arbeitsplatz finalisieren

- Replay-Quellen sauber einbinden
- Vorschau und Program-Out definieren
- Übergabe an Bildmischer testen

---

## Grundsatz für das Projekt

Dieses Projekt soll **einfach, robust und verständlich** bleiben.

Keine überladene Broadcast-Software.
Keine unnötige Plattform.
Keine komplexe Verteilung, solange ein klarer lokaler Operator-Arbeitsplatz ausreicht.

Der Fokus ist:

- stabile Live-Kreuzschiene
- lokale Replay-Arbeitsstation
- einfache Bedienung mit N3
- OBS als sichtbarer Arbeitsplatz

---

## Kurzform in einem Satz

**Haupt-MediaMTX macht live, lokales MediaMTX macht replay, der Replay-Controller steuert, das N3 bedient, OBS zeigt und spielt aus.**

