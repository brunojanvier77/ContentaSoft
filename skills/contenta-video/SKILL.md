---
name: contenta-video
description: Compress and re-encode video files with the VideoRecompress Studio CLI (videorecompress). Use when the user asks to shrink videos, convert to H.265/HEVC, AV1 or VP9, prepare videos for WhatsApp, Discord or email, archive phone, wedding, dashcam or screen recordings, batch-compress a folder, or watch a folder for new videos.
allowed-tools: Bash
---

# VideoRecompress Studio (video compression)

Use the `videorecompress` CLI (VideoRecompress Studio 2026.2.19+, Windows). Default per-user install: `%LOCALAPPDATA%\Programs\VideoRecompressStudio\videorecompress.exe`, on the user PATH. Check with `videorecompress --version`.

**Always pass full paths** for inputs and `--output`. Version 2026.2.19 resolves relative paths against its tools folder: `analyze clip.mp4` then reports 0x0 and `recompress` exits with 4.

Every command accepts `--json` (JSON on stdout, logs on stderr), `--quiet`, `-v`.

## Commands

```bash
videorecompress analyze <full-path> [--json]
videorecompress recompress <full-path> --output <dir> [--preset-id ID | --codec C --crf N] [options]
videorecompress batch <dir> --output <dir> [--preset-id ID | --codec C --crf N] [--workers 2] [--include "*.mp4"]
videorecompress presets [--json] [--preset-id ID]
videorecompress profile save|load|validate <file.json>
videorecompress watch --folder <dir> --preset <ID> [--output <dir>]     # runs until Ctrl+C, scans every 30 s
videorecompress register <email> <key>
videorecompress status
```

`--output` is a folder; outputs keep the source name.

## Presets: two different flags

- `--preset-id <id>` = named preset (recompress, batch, profile save). Explicit flags override single settings.
- `--encoder-preset` = encoder speed (`ultrafast`..`veryslow`, or 0-13 for SVT-AV1; default `medium`). There is no `--preset` on recompress/batch.
- `watch` uses `-p/--preset <id>` for the named preset.

| Goal | `--preset-id` | Codec / CRF |
|------|---------------|-------------|
| Phone/home video archive | `phone_archive` | H.265 / 23 |
| Wedding/event archive (high quality) | `wedding_archive` | H.265 / 18 |
| Maximum savings | `max_savings` | AV1 / 35 |
| Surveillance | `security_archive` | H.265 / 28 |
| Dashcam / body cam | `dashcam_archive` | H.265 / 27 |
| Screen recordings | `screen_recording` | H.265 / 26 |
| Fast, plays everywhere | `quick_h264` | H.264 / 23 |
| Web (WebM) | `web_optimized` | VP9 / 30 |
| 4K down to 1080p | `4k_to_1080p` | H.265 / 22 |
| WhatsApp / email / Discord | `whatsapp`, `email_attachment`, `discord_free` | H.264 / 28-32 |

Full list: `videorecompress presets --json` (24 presets).

## Other options (recompress and batch)

`--quality-mode crf|cbr|twopass` + `--bitrate kbps` · `--hw-accel auto|nvenc|qsv|amf|software` · `--audio-mode copy|reencode|remove` · `--audio-codec aac|opus|mp3|flac|ac3|eac3` · `--audio-bitrate` · `--container mp4|mkv|webm` · `--width/--height` · `--denoise off|light|medium|strong` · `--deinterlace off|yadif|yadif-bob` · `--watermark-text/--watermark-image` + `--watermark-position TopLeft..BottomRight` · `--fps/--fps-mode` · `--overwrite` · `--profile file.json` · `--measure-quality` (SSIM/VMAF). batch: `-r/--recursive` (default on), `-w/--workers` (default 2), `--include/--exclude`.

Do not use `--trim-mode` in 2026.2.19: the output check compares against the untrimmed length and the command exits with 4.

## Examples (verified on 2026.2.19)

```powershell
videorecompress analyze C:\Videos\video.mp4 --json
videorecompress recompress C:\Videos\video.mp4 --output C:\Compressed
videorecompress recompress C:\Videos\video.mp4 --codec h265 --crf 18 --encoder-preset slow --output C:\Archive
videorecompress batch C:\Videos --preset-id phone_archive --workers 2 --output C:\Compressed
videorecompress batch C:\Videos --codec av1 --crf 35 --output C:\Compressed --json
videorecompress watch --folder C:\Incoming --preset phone_archive --output C:\Compressed
```

`batch --json` prints one JSON object per line: `{"type":"progress",...}` objects, then a summary `{"success":true,"cancelled":false,"totalFiles":N,"processed":N,"failed":0,"skipped":0,"durationMs":...,"errors":[]}`.

## Exit codes

0 success · 1 general error · 2 invalid arguments · 3 trial ended · 4 recompression failed · 5 file not found · 6 ffmpeg/ffprobe missing.

## Guidelines

- Run `analyze` first; report size before and after.
- A source that is already small or low-resolution can come out larger (the CLI prints "This file came out larger than the original"). Social presets with a fixed height can enlarge small videos; for those, compare sizes before replacing anything.
- Hardware encoding is automatic; `--hw-accel software` forces the CPU.
- Trial: 30 days; the first 10 files are unrestricted, then output is watermarked and cut at 10 minutes. After 30 days `recompress`, `batch` and `serve` exit with code 3 until `videorecompress register <email> <key>`.
