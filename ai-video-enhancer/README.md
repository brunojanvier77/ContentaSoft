# AI Video Enhancer Studio

AI-powered video enhancement for Windows. Upscale, interpolate frames, stabilize, and denoise videos using NVIDIA Video Super Resolution (VSR) and RIFE.

[Download Free Trial](https://contenta-software.com/aivideoenhancer/)

## Features

- **AI upscaling** — NVIDIA NGX Video Super Resolution (VSR) 2x/3x/4x, plus same-resolution `enhance` cleanup (NVIDIA RTX GPUs only)
- **Frame interpolation** — RIFE AI to multiply the frame rate 2x/4x (e.g. 30fps to 60fps)
- **Stabilization** — FFmpeg vidstab for shaky footage
- **Denoising** — nlmeans/hqdn3d noise reduction
- **Deinterlacing & sharpening** — yadif/yadifbob/bwdif deinterlace, adjustable sharpen
- **GPU acceleration** — NVIDIA NVENC, Intel QSV, AMD AMF for encoding (CPU fallback)
- **8 presets** — old video restoration, surveillance, content creation, drone/action cam, animation, archival, enhance-only cleanup, smooth motion

## GPU Requirements

| Feature | Requirement |
|---------|-------------|
| AI upscaling (VSR) | NVIDIA RTX GPU with a recent driver — **NVIDIA only**, no CPU fallback |
| Frame interpolation (RIFE) | Vulkan-capable GPU (NVIDIA or AMD) |
| Video encoding | NVIDIA NVENC, Intel QSV, or AMD AMF — falls back to CPU encoding automatically |
| Stabilize / denoise / deinterlace / sharpen | No GPU required (FFmpeg) |

Run `aivideoenhancer status` to see what your system supports.

## Install & PATH

The app installs to `C:\Program Files\ContentaSoft\AI Video Enhancer Studio\`. The CLI executable is `aivideoenhancer.exe` in that directory — add it to your `PATH`, or call it with the full path:

```bash
"C:\Program Files\ContentaSoft\AI Video Enhancer Studio\aivideoenhancer.exe" status
```

If `aivideoenhancer status` prints your GPU and tool availability, you're ready.

## CLI Reference

**Executable**: `aivideoenhancer`

| Command | Description |
|---------|-------------|
| `enhance <input>` | Enhance a video file or every video in a directory |
| `analyze <input>` | Analyze video properties |
| `presets` | List available enhancement presets |
| `status` | Check GPU capabilities, tools, and trial/registration status |
| `diagnose` | Create a diagnostic bundle for support |
| `serve` | Start the MCP server (stdio) |

### `enhance` options

| Option | Values | Description |
|--------|--------|-------------|
| `-o`, `--output` | directory | Output **directory** (not a filename) |
| `--preset` | preset id | Apply a preset (see below) |
| `--upscale` | `off`, `enhance`, `x2`, `x3`, `x4` | AI upscale; `enhance` = clean up at same resolution |
| `--denoise` | `off`, `light`, `medium`, `strong` | Noise reduction level |
| `--denoise-method` | `nlmeans`, `hqdn3d` | Denoise filter (default: `hqdn3d`) |
| `--stabilize` | `off`, `light`, `medium`, `strong` | Stabilization strength |
| `--deinterlace` | `off`, `yadif`, `yadifbob`, `bwdif` | Deinterlace filter |
| `--sharpen` | `off`, `light`, `medium`, `strong` | Sharpen strength |
| `--interpolate` | `off`, `x2`, `x4` | RIFE frame interpolation (multiplies fps) |
| `--codec` | `h264`, `h265`, `av1`, `vp9` | Output codec |
| `--crf` | 1–51 | Quality (lower = better) |
| `--upscale-tile` | size | Manual upscale tile size override (0 = auto) |
| `--frame-batch` | count | Frames per AI-runner invocation (default: 50) |

### Exit codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | License required (trial expired) |
| 4 | Enhancement failed |
| 5 | File not found |
| 6 | Tools missing |

## Presets

| ID | Focus | Best For |
|----|-------|----------|
| `old_video_restoration` | Stabilize + Denoise strong + Upscale 2x + Audio cleanup | Old VHS/DVD/analog footage |
| `surveillance_enhancement` | Denoise + Upscale 4x | Security cameras |
| `content_creation` | Upscale 2x + Interpolate x2 | YouTube/social |
| `drone_action_cam` | Strong stabilization + Upscale 2x + Interpolate x2 | Action/drone footage |
| `animation_anime` | Denoise light + Upscale 4x | Animated content |
| `video_archival` | Stabilize + Denoise + Upscale 2x (H.265) | Long-term storage |
| `enhance_cleanup` | Denoise + same-resolution VSR cleanup | Compression artifacts, no resolution change |
| `smooth_motion` | Interpolate x2 only (no upscale) | Fluid motion at 2x the frame rate |

All presets use the same NVIDIA VSR upscaler — there is no separate anime model. No preset targets an exact fps; interpolation multiplies the source frame rate (x2/x4).

## Quick Start

```bash
# 1. Check GPU and tool availability first
aivideoenhancer status

# 2. Restore old footage with a preset (--output is a DIRECTORY)
aivideoenhancer enhance "C:\Videos\old-tape.mp4" --output "C:\Videos\restored" --preset old_video_restoration

# 3. Upscale 2x with light denoise
aivideoenhancer enhance "C:\Videos\clip.mp4" -o "C:\Videos\enhanced" --upscale x2 --denoise light --codec h265 --crf 18

# 4. Batch-enhance a whole folder (smooth 30fps -> 60fps)
aivideoenhancer enhance "C:\Videos\incoming" -o "C:\Videos\smooth" --interpolate x2
```

## Trial

The free trial runs for 30 days. The first **5 files** (lifetime) are enhanced without branding; after that, trial output gets a watermark and is capped at 1280x720 until you register.

## MCP Server

AI Video Enhancer includes a built-in MCP server with 4 tools for AI agent integration. See the [full MCP documentation](mcp-server.md).

```bash
aivideoenhancer serve
```

Tools: `analyze_video`, `enhance_video`, `list_presets`, `get_status`. The MCP server works during an active trial and is blocked once the trial expires (exit code 3).
