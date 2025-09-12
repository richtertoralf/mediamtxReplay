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

```bash
# paar Variablen setzen
HOST=127.0.0.1; PORT=9996; STREAM=cam1
```

alle Segmente (Start .. Ende)
```bash
curl -fsSg "http://${HOST}:${PORT}/list" --data-urlencode "path=${STREAM}" \
| jq -r '.segments[]? | "\(.start) .. \(.end)"'

```

Letzten Eintrag (jüngstes Segment) zeigen:
```bash
curl -fsSg "http://${HOST}:${PORT}/list" --data-urlencode "path=${STREAM}" \
| jq -r '(.segments | last)? | select(.) | "\(.start) .. \(.end)"'

```

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

