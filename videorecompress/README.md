# VideoRecompress Studio

Desktop video compression tool for Windows. Reduce video file sizes by 30-70% using modern codecs with GPU acceleration.

**$49 Single / $79 Family / $149 Commercial** | [Download Free Trial](https://contenta-videorecompress.com)

## Features

- **Modern codecs** — H.264, H.265 (HEVC), AV1, VP9
- **GPU acceleration** — NVIDIA NVENC, Intel QSV, AMD AMF (10x faster)
- **7 smart presets** — from quick H.264 to max-savings AV1
- **Batch processing** — process entire folders
- **Video analysis** — detailed codec/bitrate/resolution info via ffprobe
- **Folder watching** — auto-compress new files as they appear
- **30-70% savings** — typical space reduction with no visible quality loss

## CLI Reference

**Executable**: `videorecompress`

**Global options**: `--json`, `--quiet`, `-v`/`--verbose`

| Command | Description |
|---------|-------------|
| `recompress` | Recompress a single video |
| `batch` | Batch recompress a directory |
| `analyze` | Analyze video properties and estimate savings |
| `presets` | List available presets |
| `profile` | Show user/license info |
| `status` | Check trial/registration status |
| `watch` | Watch folder for auto-compression |

## Presets

| ID | Codec | CRF | Target | Est. Savings |
|----|-------|-----|--------|-------------|
| `phone_archive` | H.265 | 23 | Home video | 40-50% |
| `youtube_raw` | H.265 | 20 | Content creators | 30-40% |
| `security_archive` | H.265 | 28 | Surveillance | 50-60% |
| `wedding_archive` | H.265 | 18 | Videographers | 25-35% |
| `max_savings` | AV1 | 35 | Maximum compression | 55-70% |
| `quick_h264` | H.264 | 23 | Fast, compatible | 20-30% |
| `web_optimized` | VP9 | 30 | Web delivery | 45-55% |

## CLI Examples

### Analyze a video

```bash
videorecompress analyze video.mp4 --json
```

### Recompress with a preset

```bash
videorecompress recompress video.mp4 --output compressed.mp4 --preset phone_archive
```

### Batch compress a folder

```bash
videorecompress batch --input ./videos --output ./compressed --preset max_savings
```
