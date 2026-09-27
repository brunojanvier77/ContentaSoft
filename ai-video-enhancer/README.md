# AI Video Enhancer Studio

Upscale, stabilize, denoise and smooth old or low-quality videos on Windows, and cut highlight reels and still frames from them. The same install gives you the desktop app, the `aivideoenhancer` CLI and an MCP server for AI agents.

[Download the free trial](https://www.contenta-software.com/aivideoenhancer/) | [MCP server reference](mcp-server.md)

Documented version: **2026.7.13**.

## What it does

- **Upscaling** x2, x3 or x4 with NVIDIA Video Super Resolution (VSR), or `enhance` to clean up at the same resolution.
- **Frame interpolation** with RIFE up to a target of 30 or 60 fps.
- **Stabilization**, **rolling-shutter (jello) correction**, **denoise**, **deinterlace**, **sharpen**.
- **Before/after comparison video** next to each output (`--compare`).
- **Still frames**: evenly spaced, at given times, one per scene, optionally upscaled with VSR (`extract-frames`).
- **Highlight reels**: suggest clips by scene changes and loudness, then render them into one MP4 with optional music (`remix`).
- **8 presets** for common jobs (old tapes, surveillance, social media, drone footage, animation, archiving).

## GPU requirements

| Feature | Needs |
|---------|-------|
| VSR upscaling (`--upscale x2/x3/x4/enhance`) | NVIDIA RTX GPU with a recent driver. No CPU fallback |
| Frame interpolation (`--interpolate`) | Vulkan-capable GPU |
| Encoding | NVIDIA NVENC, Intel QSV or AMD AMF when available, CPU otherwise |
| Stabilize, rolling shutter, denoise, deinterlace, sharpen | No GPU needed |

`aivideoenhancer status` shows your GPU, VRAM, Vulkan support and encoders.

## Install and PATH

The CLI is installed with the desktop app. A default install is per-user, in `%LOCALAPPDATA%\Programs\AIVideoEnhancerStudio\`, and the installer adds that folder to your user `PATH`. Open a new terminal after installing, then check:

```powershell
aivideoenhancer --version    # 2026.7.13 or later
aivideoenhancer status
```

If `aivideoenhancer` is not found, or `--version` prints an older number, call the exe by its full path or remove the older copy that comes first on `PATH`.

The CLI prints informational log lines (GPU detection, progress details) along with its normal output. There is no `--json` flag except on `remix suggest`; for structured results use the MCP server.

## Commands

| Command | What it does |
|---------|--------------|
| `enhance <input>` | Enhance one video, or every video in a folder |
| `extract-frames <input>` | Save still frames from a video or a folder of videos |
| `remix suggest <input>` | Propose highlight clip ranges |
| `remix render <input>` | Render clip ranges into one H.264 MP4 |
| `analyze <input>` | Codec, resolution, frame rate, duration, bitrate, audio |
| `presets` | List the enhancement presets |
| `status` | GPU, encoders, license state |
| `diagnose` | Write a diagnostic zip for support |
| `serve` | Start the MCP server (stdio) |

There is no `register` command in the CLI; register in the desktop app.

## Examples

Each of these was run against 2026.7.13.

```powershell
# 2x upscale with light denoise, H.265 at CRF 18
aivideoenhancer enhance .\clip.mp4 -o .\enhanced --upscale x2 --denoise light --codec h265 --crf 18

# Smooth motion to 60 fps, plus a side-by-side before/after video
aivideoenhancer enhance .\clip.mp4 -o .\smooth --interpolate 60 --compare

# A whole folder of old tapes with a preset
aivideoenhancer enhance C:\Videos\tapes -o C:\Videos\restored --preset old_video_restoration

# Run it again later: skip videos that already have an output
aivideoenhancer enhance C:\Videos\tapes -o C:\Videos\restored --preset old_video_restoration --skip-existing

# 5 thumbnail candidates as PNG
aivideoenhancer extract-frames video.mp4 -o .\frames --preset thumbnails

# 6 evenly spaced JPEGs, each the sharpest of 5 nearby candidates
aivideoenhancer extract-frames video.mp4 -o .\frames6 --count 6 --format jpg --quality 90 --sharpest

# Frames at 2 s, 5 s and 10 s, upscaled 2x
aivideoenhancer extract-frames video.mp4 -o .\frames_up --timestamps 2,5,10 --upscale x2

# Suggest highlight clips (text, or --json)
aivideoenhancer remix suggest video.mp4 --duration 10
aivideoenhancer remix suggest video.mp4 --duration 10 --json

# Render three clips as a vertical reel with music ducked under the original sound
aivideoenhancer remix render video.mp4 --clips "1-4,8-11,14-17" -o .\reel.mp4 --aspect 9:16 --music music.mp3 --duck
```

`enhance` writes `<name>_enhanced.mp4` into the output folder (`-o` is a folder), and `<name>_enhanced_compare.mp4` with `--compare`. `remix suggest` ends with a ready-to-run `remix render` command.

## enhance options

| Option | Values |
|--------|--------|
| `-o, --output` | Output folder |
| `--preset` | A preset ID (below); other options override its values |
| `--upscale` | `off`, `enhance` (same resolution), `x2`, `x3`, `x4` |
| `--denoise` | `off`, `light`, `medium`, `strong` |
| `--denoise-method` | `nlmeans`, `hqdn3d` (default `hqdn3d`) |
| `--stabilize` | `off`, `light`, `medium`, `strong` |
| `--rolling-shutter` | `off`, `light`, `medium`, `strong` |
| `--deinterlace` | `off`, `yadif`, `yadifbob`, `bwdif` |
| `--sharpen` | `off`, `light`, `medium`, `strong` |
| `--interpolate` | `off`, `30`, `60`: target frame rate. Only raises it; a clip already at or above the target is left as is |
| `--codec` | `h264`, `h265`, `av1`, `vp9` (default `h265`) |
| `--crf` | 1-51, lower = better |
| `--frame-batch` | Frames per upscaling pass; 0 = automatic from free VRAM (default) |
| `--upscale-tile` | Accepted for older scripts; ignored |
| `--temp-dir` | Scratch folder for extracted frames (default: the app's setting, or the system temp folder) |
| `--skip-existing` | Leave a video alone if its output already exists (default: write a renamed copy). Skipped videos do not use a free trial file |
| `--compare` | Also write a before/after video |
| `--compare-labels` | `"FIRST\|SECOND"` (default `"BEFORE\|AFTER"`) |
| `--compare-layout` | `horizontal` (default, side by side) or `vertical` (stacked) |

## extract-frames options

| Option | Values |
|--------|--------|
| `-o, --output` | Output folder (required). A `--preset` run writes into `<output>\<name>_frames\` |
| `--preset` | `storyboard`, `thumbnails`, `social`, `scenes` (one frame per scene change), `custom` |
| `--count` | Number of evenly spaced frames; with `--preset scenes`, the top N scenes |
| `--timestamps` | Comma-separated seconds |
| `--upscale` | `off` (default), `enhance`, `x2`, `x3`, `x4` |
| `--auto-upscale-below` | Use x2 when the video height is below this value (0 = off) |
| `--format` | `png` (default), `jpg`, `webp` |
| `--quality` | 1-100 for jpg/webp (default 90) |
| `--sharpest` | Pick the sharpest of 5 nearby candidates per frame |

## remix options

`remix suggest`: `--duration` (target total seconds, default 30), `--clips` (count; default 5-8 from the duration), `--min-clip` / `--max-clip` (default 3 / 6 s), `--threshold` (scene sensitivity 0-100, default 10), `--audio-weight` (0-1, how much loud moments count), `--window-start` / `--window-end` (required for videos over 4 hours), `--json`.

`remix render`: `--clips "start-end,..."` and `-o <file.mp4>` (both required), `--max-duration` (default 30 s), `--mute`, `--width`, `--aspect original|9:16|1:1|16:9`, `--music <file>`, `--music-volume` (default 30), `--duck`.

## Presets

| ID | Pipeline |
|----|----------|
| `old_video_restoration` | Stabilize medium, denoise strong, upscale x2, audio cleanup |
| `surveillance_enhancement` | Denoise medium, upscale x4, light audio cleanup |
| `content_creation` | Denoise light, upscale x2, interpolate to 30 fps |
| `drone_action_cam` | Stabilize strong, denoise light, upscale x2, interpolate to 30 fps |
| `animation_anime` | Denoise light, upscale x4 |
| `video_archival` | Stabilize light, denoise medium, upscale x2 |
| `enhance_cleanup` | Denoise light, same-resolution cleanup |
| `smooth_motion` | Denoise light, interpolate to 30 fps |

All presets use the same NVIDIA upscaler; there is no separate anime model.

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | License required: the trial has ended (`serve`) |
| 4 | Enhancement failed |
| 5 | File not found |
| 6 | Tools missing |

## Trial and license

- The trial lasts 30 days. The first **5 files** come out at full resolution without a watermark. After that, or after 30 days, enhanced videos carry a watermark and are capped at 1280x720 until you register.
- A remix render uses one free file (the reel itself is never watermarked). `extract-frames` with upscaling uses one free file per video; once they are used up, stills are still extracted, without the upscale.
- After 30 days the MCP server refuses to start (exit code 3).
- License: **$129 one-time** for a lifetime license; the [buy page](https://www.contenta-software.com/aivideoenhancer/buy.php) also lists a quarterly plan. Register in the desktop app.

## MCP server

`aivideoenhancer serve` starts an MCP server with 4 tools: `analyze_video`, `enhance_video`, `list_presets`, `get_status`. See the [MCP reference](mcp-server.md).
