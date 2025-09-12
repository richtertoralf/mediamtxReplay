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
  - Zweiter Klick: gewünschte Replay-Dauer (10 s, 20 s, 40 s).  
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
