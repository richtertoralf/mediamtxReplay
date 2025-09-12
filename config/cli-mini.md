# MediaMTX Replay – CLI Mini

## 1) Vorhandene Streams/Pfade (Record-Ordner)
> Pfad aus deiner Config: `recordPath: /var/lib/mediamtx/recordings/%path/...`

```bash
# nur die Pfad-Ordner (eine Ebene)
ls -1 /var/lib/mediamtx/recordings

# alternativ inkl. Zeitstempel-Unterordner flach (nur Namen)
find /var/lib/mediamtx/recordings -maxdepth 2 -type d -printf "%P\n" | sed '/^$/d'
```

## 2) /list als Einzeiler mit jq

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

