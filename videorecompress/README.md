# VideoRecompress Studio

Desktop video compression tool for Windows. Reduce video file sizes by 30-70% using modern codecs with GPU acceleration.

[Download Free Trial](https://www.contenta-software.com/videorecompress/)

## Features

- **Modern codecs** — H.264, H.265 (HEVC), AV1, VP9
- **GPU acceleration** — NVIDIA NVENC, Intel QSV, AMD AMF (10x faster)
- **24 smart presets** — home video, social media, surveillance, gaming, and more
- **Batch processing** — process entire folders
- **Video analysis** — detailed codec/bitrate/resolution info via ffprobe
- **Folder watching** — auto-compress new files as they appear
- **30-70% savings** — typical space reduction with no visible quality loss

## Install & PATH

The app installs to `C:\Program Files\ContentaSoft\VideoRecompress Studio\`. The CLI executable is `videorecompress.exe` in that folder — add the folder to your `PATH`, or call it with the full path:

```bash
"C:\Program Files\ContentaSoft\VideoRecompress Studio\videorecompress.exe" --help
```

Verify the install (shows ffmpeg/ffprobe availability, license state, and GPU encoder support):

```bash
videorecompress status
```

## Quick Start

```bash
# 1. See what you're working with
videorecompress analyze video.mp4

# 2. Compress with the defaults (H.265, CRF 23) into C:\Compressed
videorecompress recompress video.mp4 --output C:\Compressed

# 3. Archive-quality H.265 (same settings as the "Wedding / Event Archive" preset)
videorecompress recompress video.mp4 --codec h265 --crf 18 --output C:\Compressed

# 4. Batch compress using a named preset (same settings as the GUI "Phone Video Archive" tile)
videorecompress batch C:\Videos --preset-id phone_archive --workers 4 --output C:\Compressed

# 5. Batch compress a whole folder to AV1 with explicit flags and 4 parallel workers
videorecompress batch C:\Videos --codec av1 --crf 35 --workers 4 --output C:\Compressed
```

Note: `--output` takes a **directory**, not a filename. Output files keep the source name.

## CLI Reference

**Executable**: `videorecompress`

**Global options** (all commands): `--json`, `--quiet`, `-v`/`--verbose`

| Command | Description |
|---------|-------------|
| `analyze <file>` | Show video file information (codec, resolution, duration, etc.) |
| `recompress <input>` | Recompress a single video file |
| `batch <input-dir>` | Batch recompress all videos in a directory |
| `presets` | List available recompression presets (`--preset-id <id>` for details) |
| `register <email> <key>` | Register a license key |
| `profile save\|load\|validate <file>` | Save, load, or validate recompression profiles (JSON) |
| `status` | Show tool availability, license state, and GPU info |
| `watch` | Monitor a folder and auto-recompress new video files |
| `serve` | Start the MCP server (stdio) |

### recompress / batch options

Both commands share the same encoding options:

| Option | Values | Default |
|--------|--------|---------|
| `--codec` | `h264`, `h265`, `av1`, `vp9`, `copy` | `h265` |
| `--crf` | 0-63 (lower = better quality) | `23` |
| `--quality-mode` | `crf`, `cbr`, `twopass` | `crf` |
| `--bitrate` | target kbps (for cbr/twopass) | — |
| `--preset` | **encoder speed**: `ultrafast`..`veryslow` (numeric for AV1) | `medium` |
| `--preset-id` | **named preset** to use as base settings (e.g. `phone_archive`); explicit flags override | — |
| `--hw-accel` | `auto`, `nvenc`, `qsv`, `amf`, `software` | `auto` |
| `--audio-mode` | `copy`, `reencode`, `remove` | `copy` |
| `--audio-codec` | `aac`, `opus`, `mp3`, `flac`, `ac3`, `eac3` | `aac` |
| `--audio-bitrate` | kbps | `128` |
| `--container` | `mp4`, `mkv`, `webm` | `mp4` |
| `--width` / `--height` | pixels (0 = keep original) | `0` |
| `--denoise` | `off`, `light`, `medium`, `strong` | `off` |
| `--deinterlace` | `off`, `yadif`, `yadif-bob` | `off` |
| `--watermark-text` / `--watermark-image` | text or PNG/JPG path | — |
| `--watermark-position` / `--watermark-opacity` / `--watermark-scale` | position, 0-100, % of width | `BottomRight`, `70`, `25` |
| `--trim-mode` | `none`, `skip-start`, `skip-end`, `keep-first`, `custom` | `none` |
| `--trim-start` / `--trim-end` / `--trim-duration` | seconds | `0` |
| `--fps` / `--fps-mode` | target fps / `auto`, `cfr`, `vfr` | — / `auto` |
| `--output` | output **directory** | source dir |
| `--overwrite` | overwrite existing outputs | off |
| `--profile <file>` | load settings from a JSON profile | — |
| `--measure-quality` | measure SSIM/VMAF after encoding | off |

`batch` extras: the input directory is a **positional argument** (no `--input` flag), plus `--include`/`--exclude` glob patterns, `-r`/`--recursive` (default: true), and `-w`/`--workers` (default: 2).

## How Presets Work from the CLI

This trips people up, so read carefully:

- On `recompress` and `batch`, **`--preset` is the encoder speed preset** (`ultrafast`..`veryslow`, or numeric for AV1; default `medium`). It is *not* a named recompression preset — `recompress video.mp4 --preset wedding_archive` will **fail**.
- Use **`--preset-id <id>`** on `recompress`, `batch`, and `profile save` to start from one of the 24 named presets below. Any explicit flags you also pass (e.g. `--crf 20`) override that preset's value for that setting only.
- `watch` uses **`--preset`** (not `--preset-id`) for the named preset ID — same IDs, different flag name.
- The MCP tools (`recompress_video`, `batch_recompress`, `estimate_savings`) accept a `preset` parameter with the same IDs.

```bash
# Named preset on recompress/batch
videorecompress recompress video.mp4 --preset-id phone_archive --output C:\Compressed
videorecompress batch C:\Videos --preset-id wedding_archive --workers 2 --output C:\Compressed

# Or save a preset to a JSON profile for reuse
videorecompress profile save wedding.json --preset-id wedding_archive
videorecompress recompress video.mp4 --profile wedding.json --output C:\Compressed
```

## Presets

List them with `videorecompress presets`; show one in detail with `videorecompress presets --preset-id wedding_archive`.

### Compression

| ID | Codec | CRF | Target | Est. Savings |
|----|-------|-----|--------|-------------|
| `phone_archive` | H.265 | 23 | Home video | 40-50% |
| `youtube_raw` | H.265 | 20 | Content creators | 30-40% |
| `security_archive` | H.265 | 28 | Surveillance | 50-60% |
| `wedding_archive` | H.265 | 18 | Videographers | 25-35% |
| `max_savings` | AV1 | 35 | Maximum compression | 55-70% |
| `av1_max_savings` | AV1 | 33 | Maximum compression (slower) | 60-75% |
| `quick_h264` | H.264 | 23 | Fast, compatible | 20-30% |
| `web_optimized` | VP9 | 30 | Web delivery | 45-55% |
| `drone_footage` | H.265 | 21 | Drone & action cam | 50-65% |
| `screen_recording` | H.265 | 26 | Screen recordings & tutorials | 55-70% |
| `4k_to_1080p` | H.265 | 22 | 4K downscale to 1080p | 65-80% |
| `dashcam_archive` | H.265 | 27 | Dashcam & body cam | 50-60% |
| `gaming_clips` | H.265 | 22 | Gaming recordings | 45-60% |
| `old_video_rescue` | H.265 | 20 | Legacy video conversion | 40-60% |

### Social media

| ID | Codec | CRF | Target | Est. Savings |
|----|-------|-----|--------|-------------|
| `whatsapp` | H.264 | 28 | WhatsApp / messaging | 60-75% |
| `email_attachment` | H.264 | 28 | Email attachments | 55-70% |
| `discord_free` | H.264 | 32 | Discord free tier (8 MB) | 70-85% |
| `twitter_x` | H.264 | 23 | Twitter / X | 30-45% |
| `instagram_reels` | H.264 | 23 | Instagram Reels | 30-45% |
| `tiktok` | H.264 | 23 | TikTok | 30-45% |

### Tools (no re-encoding)

| ID | Purpose |
|----|---------|
| `gif_creator` | Create GIFs from video |
| `youtube_thumbnails` | Extract thumbnail frames |
| `video_contact_sheet` | Generate a contact sheet |

### Conversion

| ID | Codec | CRF | Target | Est. Savings |
|----|-------|-----|--------|-------------|
| `iphone_to_mp4` | H.264 | 20 | iPhone to MP4 | 10-25% |

## Watch Folder

`watch` monitors a folder and auto-compresses new videos using a **named preset ID** (this is where named presets work directly on the CLI):

```bash
videorecompress watch --folder C:\Videos --preset phone_archive --output C:\Compressed
```

`-f`/`--folder` and `-p`/`--preset` are required; `-o`/`--output` defaults to the source folder; `-r`/`--recursive` defaults to true. Watermark flags are also available.

## JSON Output

Every command supports `--json` for machine-readable output on **stdout**. Diagnostic logs go to **stderr**, so piping stdout is safe:

```bash
videorecompress analyze video.mp4 --json | jq .videoCodec
```

`batch --json` emits **NDJSON** (one JSON object per line): a progress object per file event (`"type": "progress"`), followed by a final summary object:

```
{"type":"progress","file":"a.mp4","status":"completed",...}
{"type":"progress","file":"b.mp4","status":"completed",...}
{"total":2,"success":2,"failed":0,...}
```

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | License required (trial expired — `recompress`/`batch`/`serve` only) |
| 4 | Recompression failed |
| 5 | File not found |
| 6 | Tools missing (ffmpeg/ffprobe not found — run `videorecompress status` to troubleshoot) |

## Trial

The free trial lasts 30 days. Your first **3 files (lifetime)** are compressed free with no restrictions; from the 4th file, output gets a watermark and is capped at 600 seconds (10 minutes). After 30 days, `recompress`, `batch`, and `serve` are blocked (exit code 3) until you register:

```bash
videorecompress register your@email.com XXXXX-XXXXX-XXXXX-XXXXX-XXXXX
```

Registration also works in the desktop app's Register page. `analyze`, `presets`, `status`, `profile`, `register`, and `watch` setup work without a license check.

## MCP Server

VideoRecompress includes a built-in MCP server with 5 tools for AI agent integration. See the [full MCP documentation](mcp-server.md).

```bash
videorecompress serve
```

Tools: `analyze_video`, `recompress_video`, `list_presets`, `estimate_savings`, `batch_recompress`.
