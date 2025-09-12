# MediaMTX Replay – Settings

Dieses Dokument beschreibt ausschließlich die **Replay-relevanten Einstellungen** in  
`/usr/local/etc/mediamtx.yml`.  

Ziel:  
- **8 Streams** (z. B. SRT/RTMP/RTSP-Ingest)  
- **ca. 10 Minuten Replay-Puffer**  
- **schnelles automatisches Löschen** alter Segmente  
- optimiert für **Sport-Produktionen** (schneller Clip-Start, saubere Seek-Punkte)

---

## Playback-Server aktivieren

Der Playback-Server stellt die Endpunkte `/list` und `/get` bereit, um zeitbasierte Clips aus Aufzeichnungen abzurufen.

```yaml
###############################################
# Global settings -> Playback server

# Enable downloading recordings from the playback server.
playback: yes
# Address of the playback server listener.
playbackAddress: :9996
```
- Port 9996 ist Standard.
- /list → zeigt verfügbare Zeitfenster
- /get → liefert Clips (fMP4 = Default, MP4 optional)

## Recording aktivieren (Default Path Settings)

Alle Streams werden automatisch aufgezeichnet. Die folgenden Parameter sind optimal für Sport-Replay:

```yaml
###############################################
# Default path settings -> Record

# Record streams to disk.
record: yes

# Speicherort & Namensschema (empfohlen: absoluter Pfad)
recordPath: /var/lib/mediamtx/recordings/%path/%Y-%m-%d_%H-%M-%S-%f

# Containerformat
recordFormat: fmp4              # optimal für Replay/Playback; Alternative: mpegts

# Fragmentierung
recordPartDuration: 300ms       # 200–500ms → schneller Clip-Start
recordMaxPartSize: 100M         # bei 4K/hoher Bitrate ggf. 150–200M

# Segment-Rotation
recordSegmentDuration: 10s      # 60–120s = guter Kompromiss (kleine Dateien, schneller verfügbar)

# Automatisches Löschen
recordDeleteAfter: 15m          # ~10 Minuten Replay-Puffer + Sicherheitsreserve

```

## Tipps für Sport-Replay

- Encoder sauber konfigurieren: konstante FPS, Keyframe-Intervall ≤ 1–2 s, CBR-Audio.
- Formatwahl: recordFormat: fmp4 für schnelle Replays, beim Abruf &format=mp4 falls ein Player zickt.
- Segmentlänge: 60–120 s sorgt für schnelle Verfügbarkeit und robustes Handling.
- Retention: recordDeleteAfter: 15m gibt dir ca. 10 m Replay-Puffer + Sicherheitsreserve.
- Speicher: SSD/NVMe mit genügend IOPS nutzen; Mount mit noatime spart I/O.
- Multi-Stream-Last (8 Kameras): Mit 8×1080p50 @ 8–12 Mbit/s rechnest du mit ~10 MB/s Schreiblast – gut handhabbar auf SSD.

### Best Practice für Sport (Skispringen, Fußball, Handball)

- recordSegmentDuration: 10s
-   Replay nach 10s garantiert abrufbar
-   Overhead im Dateisystem akzeptabel

- recordPartDuration: 300ms
-   sorgt dafür, dass Seek & Start im Clip fein genug bleiben
