# MediaMTX Replay – CLI Mini

## 1) Vorhandene Streams/Pfade (Record-Ordner)
> Pfad aus deiner Config: `recordPath: /var/lib/mediamtx/recordings/%path/...`

```bash
# nur die Pfad-Ordner (eine Ebene)
ls -1 /var/lib/mediamtx/recordings

# alternativ inkl. Zeitstempel-Unterordner flach (nur Namen)
find /var/lib/mediamtx/recordings -maxdepth 2 -type d -printf "%P\n" | sed '/^$/d'
```

## 2) /list als Einzeiler mit jq (Prinzip)

Mit dem Endpunkt `/list` fragt man die im Replay-Server gespeicherten **Segmente** eines Streams ab.  
Die Antwort ist ein JSON-Array (oder bei nur einem Segment ein einzelnes Objekt) mit Feldern:

- `start`    → Startzeitpunkt (UTC, RFC3339-Format)  
- `duration` → Länge des Segments in Sekunden (kann Nachkommastellen haben)  
- `url`      → fertiger Link zum Abruf dieses Segments über `/get`  

### Beispiel: nur die letzte URL ausgeben
```bash
# paar Variablen setzen
HOST=127.0.0.1; PORT=9996; STREAM=testpattern-clock
```

```bash
curl -sG "http://${HOST}:${PORT}/list" --data-urlencode "path=${STREAM}" \
| jq -r '.[-1].url // .url'
```

### Beispiel: Segmente aus den letzten 10 Minuten
```bash
curl -sG "http://127.0.0.1:9996/list" \
  --data-urlencode "path=testpattern-clock" \
  --data-urlencode "start=$(date -u -d '10 minutes ago' +%Y-%m-%dT%H:%M:%SZ)" \
| jq .

```
Ergebnis (Beispiel):
```json
[
  {
    "start": "2025-09-12T22:39:31Z",
    "duration": 600.595724222,
    "url": "http://127.0.0.1:9996/get?duration=600.595724222&path=testpattern-clock&start=2025-09-12T22%3A39%3A31Z"
  }
]

```
Was bedeutet das?

Das Segment hat um 22:39:31 UTC begonnen.

Es ist etwa 600 Sekunden (= 10 Minuten) lang.

Über den url-Wert kann man das Segment direkt mit /get herunterladen oder im Browser/VLC öffnen.

Mit den Parametern start= und end= lässt sich die Abfrage einschränken, z. B. nur ein bestimmter Zeitraum (z. B. Vormittag eines Wettkampftages).

Kurz gesagt

/list zeigt, was aufgezeichnet ist.

.url liefert direkt den Abruf-Link.

Mit jq kann man bequem die letzten Einträge oder bestimmte Zeitfenster filtern.

## Skript zur Anzeige der Replay-URL 20 Sekunden zurück
```bash
#!/usr/bin/env bash
# letzte 20s, minimale Variante (MediaMTX: /get?start=...&duration=...)
set -euo pipefail

HOST="$(hostname -I 2>/dev/null | awk '{print $1}')"
[[ -z "${HOST:-}" ]] && HOST="127.0.0.1"

PORT=9996
STREAM="${1:-testpattern-clock}"     # Pfad als 1. Argument; Default = testpattern-clock
FORMAT="${FORMAT:-fmp4}"             # für VLC ggf. "mp4"
DURATION="${DURATION:-20}"           # Sekunden

# Startzeit: jetzt - DURATION (UTC), RFC3339 ohne ms ist ok; mit .000Z ebenfalls ok
START=$(date -u -d "${DURATION} seconds ago" +"%Y-%m-%dT%H:%M:%S.000Z")

echo "http://${HOST}:${PORT}/get?path=${STREAM}&start=${START}&duration=${DURATION}&format=${FORMAT}"
# Optional gleich testen:
# curl -fS -o "/tmp/replay_last.${FORMAT/mp4/mp4}" "http://${HOST}:${PORT}/get?path=${STREAM}&start=${START}&duration=${DURATION}&format=${FORMAT}" && echo "OK"
```

