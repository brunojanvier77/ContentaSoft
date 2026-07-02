---
name: contenta-video
description: Recompress and optimize video files using the VideoRecompress CLI. Use when the user asks to compress videos, reduce file sizes, convert video codecs (H.265, AV1), or batch process video folders.
allowed-tools: Bash
---

# Contenta Video Processing

You have access to the `videorecompress` CLI for video compression and optimization. All commands support `--json` for structured output on stdout (logs go to stderr). Default install path: `C:\Program Files\ContentaSoft\VideoRecompress Studio\videorecompress.exe` — use the full path if it is not on PATH. Run `videorecompress status` to verify tools/license/GPU.

**IMPORTANT — two kinds of "preset":** on `recompress` and `batch`, `--preset` is the *encoder speed* (`ultrafast`..`veryslow`, default `medium`) — NOT a named preset. Use **`--preset-id <id>`** (e.g. `phone_archive`) to apply a named recompression preset; explicit flags like `--crf` override preset values. `watch` uses `--preset` for the named preset ID. You can also use `--profile <file.json>`.

## Commands

### Register a license
```bash
videorecompress register <email> <key>
```
Removes trial limits after purchase. Also available in the desktop app's Register page.

### Analyze a video
```bash
videorecompress analyze <input> --json
```
Returns codec, resolution, bitrate, duration, file size, and audio info.

### Recompress a single video
```bash
videorecompress recompress <input> --output <dir> --preset-id phone_archive
videorecompress recompress <input> --output <dir> --codec h265 --crf 23 --hw-accel auto
```
`--output` is a **directory** (default: same as source). Use `--preset-id` for a named preset, or explicit `--codec`/`--crf`. Other useful flags: `--container mp4|mkv|webm`, `--audio-mode copy|reencode|remove`, `--width`/`--height`, `--overwrite`, `--measure-quality`.

### Batch recompress
```bash
videorecompress batch <input-dir> --preset-id phone_archive --output <dir> --workers 2
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

## Named Presets

24 built-in presets (see `presets --json`). On `recompress`/`batch`/`profile save`, use `--preset-id <id>`. On `watch` and MCP tools, use `--preset` / `preset` parameter with the same IDs.

| Goal | `--preset-id` | Manual equivalent |
|------|---------------|-------------------|
| Home video archive | `phone_archive` | `--codec h265 --crf 23` |
| Best-quality archive (weddings) | `wedding_archive` | `--codec h265 --crf 18` |
| Maximum savings | `max_savings` | `--codec av1 --crf 35` |
| Surveillance/dashcam | `security_archive` | `--codec h265 --crf 28` |
| Fast, wide compatibility | `quick_h264` | `--codec h264 --crf 23` |
| Web delivery | `web_optimized` | `--codec vp9 --crf 30` |
| 4K → 1080p | `4k_to_1080p` | `--codec h265 --crf 22 --height 1080` |
| Messaging/email (small) | `whatsapp` | `--codec h264 --crf 28 --height 720` |

## Exit Codes

0 success · 1 general error · 2 invalid arguments · 3 license required (trial expired) · 4 recompression failed · 5 file not found · 6 ffmpeg/ffprobe missing (troubleshoot with `videorecompress status`).

## Guidelines

- Always run `analyze` first to understand the input before choosing settings.
- For archival, use H.265 CRF 23 (phone_archive-equivalent) as the best balance of quality and savings.
- For maximum savings where encoding time doesn't matter, use AV1 CRF 35.
- For quick results with wide compatibility, use H.264 CRF 23.
- GPU acceleration is used automatically when available (NVIDIA NVENC, Intel QSV, AMD AMF). Use `--hw-accel software` to force CPU encoding.
- Report estimated and actual savings to the user after compression.
- Trial: the first **3 files (lifetime)** are free with no restrictions; from the 4th file, output gets a watermark and a 600-second (10-minute) duration cap. After the 30-day trial, `recompress`/`batch`/`serve` fail with exit code 3 — register with `videorecompress register <email> <key>` or in the desktop app.
