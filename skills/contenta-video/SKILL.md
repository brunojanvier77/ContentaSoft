---
name: contenta-video
description: Recompress and optimize video files using the VideoRecompress CLI. Use when the user asks to compress videos, reduce file sizes, convert video codecs (H.265, AV1), or batch process video folders.
allowed-tools: Bash
---

# Contenta Video Processing

You have access to the `videorecompress` CLI for video compression and optimization. All commands support `--json` for structured output on stdout (logs go to stderr). Default install path: `C:\Program Files\ContentaSoft\VideoRecompress Studio\videorecompress.exe` — use the full path if it is not on PATH. Run `videorecompress status` to verify tools/license/GPU.

**IMPORTANT — two kinds of "preset":** on `recompress` and `batch`, `--preset` is the *encoder speed* (`ultrafast`..`veryslow`, default `medium`) — NOT a named preset. Named preset IDs like `phone_archive` only work with `watch --preset` (and the MCP tools). For `recompress`/`batch`, pass explicit flags (`--codec`, `--crf`) or `--profile <file.json>`.

## Commands

### Analyze a video
```bash
videorecompress analyze <input> --json
```
Returns codec, resolution, bitrate, duration, file size, and audio info.

### Recompress a single video
```bash
videorecompress recompress <input> --output <dir> --codec h265 --crf 23 --hw-accel auto
```
`--output` is a **directory** (default: same as source). Other useful flags: `--container mp4|mkv|webm`, `--audio-mode copy|reencode|remove`, `--width`/`--height`, `--overwrite`, `--measure-quality`.

### Batch recompress
```bash
videorecompress batch <input-dir> --output <dir> --codec h265 --crf 23 --workers 2
```
The input directory is a **positional argument** (no `--input` flag). Recursive by default (`-r`); filter with `--include "*.mp4"` / `--exclude`.

With `--json`, batch emits **NDJSON**: one progress object per line (`"type": "progress"`) as files complete, then a final summary object (total/success/failed) — parse line by line.

### List presets
```bash
videorecompress presets --json
videorecompress presets --preset-id wedding_archive
```

### Watch folder (named presets work here)
```bash
videorecompress watch --folder <dir> --preset phone_archive --output <dir>
```
`--folder` and `--preset` (a named preset ID) are required.

## Named Presets → CLI Flag Equivalents

24 built-in presets exist (see `presets --json`), but on `recompress`/`batch` you must translate them to flags:

| Goal | Flags | Equivalent preset |
|------|-------|-------------------|
| Home video archive | `--codec h265 --crf 23` | `phone_archive` |
| Best-quality archive (weddings) | `--codec h265 --crf 18` | `wedding_archive` |
| Maximum savings | `--codec av1 --crf 35` | `max_savings` |
| Surveillance/dashcam | `--codec h265 --crf 28` | `security_archive` |
| Fast, wide compatibility | `--codec h264 --crf 23` | `quick_h264` |
| Web delivery | `--codec vp9 --crf 30` | `web_optimized` |
| 4K → 1080p | `--codec h265 --crf 22 --height 1080` | `4k_to_1080p` |
| Messaging/email (small) | `--codec h264 --crf 28 --height 720` | `whatsapp` |

## Exit Codes

0 success · 1 general error · 2 invalid arguments · 3 license required (trial expired) · 4 recompression failed · 5 file not found · 6 ffmpeg/ffprobe missing (troubleshoot with `videorecompress status`).

## Guidelines

- Always run `analyze` first to understand the input before choosing settings.
- For archival, use H.265 CRF 23 (phone_archive-equivalent) as the best balance of quality and savings.
- For maximum savings where encoding time doesn't matter, use AV1 CRF 35.
- For quick results with wide compatibility, use H.264 CRF 23.
- GPU acceleration is used automatically when available (NVIDIA NVENC, Intel QSV, AMD AMF). Use `--hw-accel software` to force CPU encoding.
- Report estimated and actual savings to the user after compression.
- Trial: the first **3 files (lifetime)** are free with no restrictions; from the 4th file, output gets a watermark and a 600-second (10-minute) duration cap. After the 30-day trial, `recompress`/`batch`/`serve` fail with exit code 3 (registration is done in the GUI app).
