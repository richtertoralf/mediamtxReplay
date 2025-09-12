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
curl -sG "http://127.0.0.1:9996/list" \
  --data-urlencode "path=cam1" \
| jq -r '.segments[] | "\(.start) .. \(.end)"'

```

Letzten Eintrag (jüngstes Segment) zeigen:
```bash
curl -sG "http://127.0.0.1:9996/list" --data-urlencode "path=cam1" \
| jq -r '.segments | last | "\(.start) .. \(.end)"'
```

