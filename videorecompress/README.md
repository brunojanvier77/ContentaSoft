# VideoRecompress Studio

Shrink phone, camera, drone and screen-recording videos by re-encoding them to H.265, AV1 or VP9 on Windows, one file or a whole folder at a time. The same install gives you the desktop app, the `videorecompress` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-software.com/videorecompress/) | [MCP server reference](mcp-server.md)

Documented version: **2026.2.19**.

## What it does

- **Codecs**: H.264, H.265 (HEVC), AV1, VP9, or `copy` to keep the video stream.
- **Hardware encoding** on NVIDIA (NVENC), Intel (QSV) and AMD (AMF) when available, software encoding otherwise.
- **24 named presets** for common jobs (phone archive, wedding archive, dashcam, screen recordings, WhatsApp, Discord and more).
- **Batch** a folder with parallel workers; **watch** a folder and compress new files.
- **Checks every output** (duration, streams) before reporting success.
- **Extras**: resize, denoise, deinterlace, frame rate, trim, text or image watermark, SSIM/VMAF measurement.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\VideoRecompressStudio\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
videorecompress --version    # 2026.2.19 or later
videorecompress status       # license, bundled tools, GPU encoders
```

If `videorecompress` is not found, or `--version` prints an older number, call the exe by its full path or remove the older copy that comes first on `PATH`.

**Use full paths in 2026.2.19.** This version resolves relative input and output paths against its bundled tools folder, so `videorecompress analyze clip.mp4` reports an empty 0x0 file and `recompress` fails with exit code 4. Give full paths (`C:\Videos\clip.mp4`, or `"$PWD\clip.mp4"` in PowerShell) as in the examples below. The MCP server is not affected because its tools already take absolute paths.

## Commands

Options available on every command: `--json` (JSON on stdout; logs go to stderr), `--quiet`, `-v`/`--verbose`.

| Command | What it does |
|---------|--------------|
| `analyze <file>` | Codec, resolution, frame rate, duration, bitrate, audio |
| `recompress <input>` | Re-encode one video |
| `batch <input-dir>` | Re-encode every video in a folder |
| `presets` | List the named presets (`--preset-id <id>` for one preset's full settings) |
| `profile save\|load\|validate <file>` | Save settings to JSON and reuse them |
| `watch` | Compress new videos that appear in a folder |
| `register <email> <key>` | Register a license key |
| `status` | License state, bundled tools, GPU encoders |
| `serve` | Start the MCP server (stdio) |

## Examples

Each of these was run against 2026.2.19 with full paths.

```powershell
# What is in the file?
videorecompress analyze C:\Videos\video.mp4
videorecompress analyze C:\Videos\video.mp4 --json

# Defaults: H.265, CRF 23, output keeps the source name
videorecompress recompress C:\Videos\video.mp4 --output C:\Compressed

# Archive quality: H.265 CRF 18 with the slower encoder preset
videorecompress recompress C:\Videos\video.mp4 --codec h265 --crf 18 --encoder-preset slow --output C:\Archive

# Start from a named preset
videorecompress recompress C:\Videos\video.mp4 --preset-id phone_archive --output C:\Compressed

# A whole folder with a named preset, 2 files at a time
videorecompress batch C:\Videos --preset-id phone_archive --workers 2 --output C:\Compressed

# A whole folder to AV1, JSON progress for scripts
videorecompress batch C:\Videos --codec av1 --crf 35 --output C:\Compressed --json

# Text watermark in the top-right corner
videorecompress recompress C:\Videos\video.mp4 --watermark-text "Draft" --watermark-position TopRight --output C:\Review

# Save a preset plus your own audio settings, then reuse it
videorecompress profile save C:\Profiles\wedding.json --preset-id wedding_archive --audio-mode reencode --audio-codec aac --audio-bitrate 192
videorecompress profile validate C:\Profiles\wedding.json
videorecompress recompress C:\Videos\video.mp4 --profile C:\Profiles\wedding.json --output C:\Archive

# See one preset's settings
videorecompress presets --preset-id wedding_archive

# Compress new files dropped into a folder (Ctrl+C to stop)
videorecompress watch --folder C:\Incoming --preset phone_archive --output C:\Compressed
```

`--output` is a folder, not a file name. Outputs keep the source file name.

## Two kinds of "preset"

- `--preset-id <id>` on `recompress`, `batch` and `profile save` picks one of the named presets below. Any flag you also pass (for example `--crf 20`) overrides that one setting.
- `--encoder-preset` is the encoder speed: `ultrafast` to `veryslow`, or an SVT-AV1 number 0-13. Default `medium`.
- `watch` takes the named preset with `-p/--preset` (required).
- The MCP tools take a `preset` parameter with the same IDs.

## recompress / batch options

| Option | Values | Default |
|--------|--------|---------|
| `--codec` | `h264`, `h265`, `av1`, `vp9`, `copy` | `h265` |
| `--crf` | 0-63, lower = better quality | `23` |
| `--quality-mode` | `crf`, `cbr`, `twopass` | `crf` |
| `--bitrate` | kbps, for `cbr` / `twopass` | |
| `--encoder-preset` | `ultrafast`..`veryslow`, or 0-13 for SVT-AV1 | `medium` |
| `--preset-id` | named preset used as the base settings | |
| `--hw-accel` | `auto`, `nvenc`, `qsv`, `amf`, `software` | `auto` |
| `--audio-mode` | `copy`, `reencode`, `remove` | `copy` |
| `--audio-codec` | `aac`, `opus`, `mp3`, `flac`, `ac3`, `eac3` | `aac` |
| `--audio-bitrate` | kbps | `128` |
| `--container` | `mp4`, `mkv`, `webm` | `mp4` |
| `--width` / `--height` | pixels, 0 = keep | `0` |
| `--denoise` | `off`, `light`, `medium`, `strong` | `off` |
| `--deinterlace` | `off`, `yadif`, `yadif-bob` | `off` |
| `--watermark-text` / `--watermark-image` | text, or a PNG/JPG path | |
| `--watermark-position` | `TopLeft`, `TopCenter`, `TopRight`, `CenterLeft`, `Center`, `CenterRight`, `BottomLeft`, `BottomCenter`, `BottomRight` | `BottomRight` |
| `--watermark-opacity` / `--watermark-scale` | 0-100 / 5-100 % of the video width | `70` / `25` |
| `--trim-mode` + `--trim-start` / `--trim-end` / `--trim-duration` | `none`, `skip-start`, `skip-end`, `keep-first`, `custom`; seconds | `none` |
| `--fps` / `--fps-mode` | target frame rate / `auto`, `cfr`, `vfr` | / `auto` |
| `--overwrite` | replace an existing output | off |
| `--profile` | load settings from a JSON profile | |
| `--measure-quality` | measure SSIM/VMAF after encoding | off |

`batch` adds `--include` / `--exclude` (glob, e.g. `"*.mp4"`), `-r/--recursive` (default on) and `-w/--workers` (default 2).

`--trim-mode` fails the output check in 2026.2.19 (the check compares against the untrimmed length and exits with code 4), so trim in the desktop app for now.

## Presets

`videorecompress presets` lists them; the savings column is the app's own estimate, and real results depend on the source. A file that is already well compressed can come out larger; the CLI says so when it happens. Presets with a fixed output height (for example `whatsapp`) also enlarge videos that are smaller than that height, so check the result on small or old clips.

| ID | Codec | CRF | Est. savings |
|----|-------|-----|--------------|
| `phone_archive` | H.265 | 23 | 40-50% |
| `youtube_raw` | H.265 | 20 | 30-40% |
| `security_archive` | H.265 | 28 | 50-60% |
| `wedding_archive` | H.265 | 18 | 25-35% |
| `max_savings` | AV1 | 35 | 55-70% |
| `av1_max_savings` | AV1 | 33 | 60-75% |
| `quick_h264` | H.264 | 23 | 20-30% |
| `web_optimized` | VP9 | 30 | 45-55% |
| `drone_footage` | H.265 | 21 | 50-65% |
| `screen_recording` | H.265 | 26 | 55-70% |
| `4k_to_1080p` | H.265 | 22 | 65-80% |
| `dashcam_archive` | H.265 | 27 | 50-60% |
| `gaming_clips` | H.265 | 22 | 45-60% |
| `old_video_rescue` | H.265 | 20 | 40-60% |
| `whatsapp` | H.264 | 28 | 60-75% |
| `email_attachment` | H.264 | 28 | 55-70% |
| `discord_free` | H.264 | 32 | 70-85% |
| `twitter_x` | H.264 | 23 | 30-45% |
| `instagram_reels` | H.264 | 23 | 30-45% |
| `tiktok` | H.264 | 23 | 30-45% |
| `iphone_to_mp4` | H.264 | 20 | 10-25% |
| `gif_creator` | tool | | |
| `youtube_thumbnails` | tool | | |
| `video_contact_sheet` | tool | | |

The last three are desktop-app tools (GIF, thumbnails, contact sheet) rather than re-encodes.

## JSON output

With `--json`, stdout carries only JSON; logs go to stderr, so piping is safe.

`batch --json` writes one JSON object per line: progress objects while it runs, then a summary.

```
{"type":"progress","file":"clip1.mp4","completed":0,"total":2,"percent":23,"status":"Encoding..."}
{"type":"progress","file":"clip2.mp4","completed":2,"total":2,"percent":0,"status":"Done"}
{"success":true,"cancelled":false,"totalFiles":2,"processed":2,"failed":0,"skipped":0,"durationMs":2226,"errors":[]}
```

## Watch folder

`watch` scans the folder every 30 seconds and compresses new videos with a named preset. `-f/--folder` and `-p/--preset` are required, `-o/--output` defaults to the source folder, `-r/--recursive` defaults to on, and the watermark options are available.

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | License required: the trial has ended (`recompress`, `batch`, `serve`) |
| 4 | Recompression failed |
| 5 | File not found |
| 6 | ffmpeg/ffprobe missing |

## Trial and license

- The trial lasts 30 days. The first **10 files** are free of restrictions; after that, output carries a watermark and is cut at 10 minutes.
- After 30 days, `recompress`, `batch` and the MCP server stop with exit code 3 until you register.
- License: **$79 one-time** for a lifetime license; the [buy page](https://www.contenta-software.com/videorecompress/buy.php) also lists higher tiers and a quarterly plan.
- Register from the CLI with `videorecompress register <email> <key>`, or in the desktop app.

## MCP server

`videorecompress serve` starts an MCP server with 5 tools: `analyze_video`, `recompress_video`, `list_presets`, `estimate_savings`, `batch_recompress`. See the [MCP reference](mcp-server.md).
