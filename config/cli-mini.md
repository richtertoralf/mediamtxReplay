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
# letzte 20s, minimale Variante
HOST="$(hostname -I | awk '{print $1}')"   # erste IP des Servers
PORT=9996
STREAM="${1:-testpattern-clock}"           # Pfad als 1. Argument; Default = testpattern-clock
FORMAT="fmp4"                              # bei VLC ggf. "mp4"

TO=$(date -u +"%Y-%m-%dT%H:%M:%S.%3NZ")
FROM=$(date -u -d '20 seconds ago' +"%Y-%m-%dT%H:%M:%S.%3NZ")

echo "http://${HOST}:${PORT}/get?path=${STREAM}&from=${FROM}&to=${TO}&format=${FORMAT}"

```

