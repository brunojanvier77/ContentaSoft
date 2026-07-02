---
name: contenta-video-enhancer
description: Enhance videos using the aivideoenhancer CLI with AI upscaling (NVIDIA Video Super Resolution), frame interpolation (RIFE), stabilization, and denoising. Use when the user asks to upscale video resolution, increase frame rate, fix shaky footage, remove noise, restore old video, or improve video quality.
allowed-tools: Bash
---

# Contenta AI Video Enhancer

You have access to the `aivideoenhancer` CLI for AI-powered video enhancement. If it is not on PATH, use the full path: `C:\Program Files\ContentaSoft\AI Video Enhancer Studio\aivideoenhancer.exe`.

**No `--json` flag** — output is human-readable text. For structured results in AI agents, use the MCP server (`aivideoenhancer serve`) or parse `status`/`analyze` text. There is no CLI `register` command yet — register in the desktop app.

## Commands

### Enhance a video
```bash
aivideoenhancer enhance <input-file-or-dir> --output <directory> --preset <preset_id>
```
Note: `--output` is a **directory**, not a filename. The input can be a single file or a folder (batch).

Or with individual settings:
```bash
aivideoenhancer enhance <input> --output <directory> --upscale x2 --denoise medium --stabilize light --codec h265 --crf 18
```

### Analyze a video
```bash
aivideoenhancer analyze <input>
```
Returns: codec, resolution, frame rate, duration, bitrate, HDR status, interlacing, audio info.

### List presets
```bash
aivideoenhancer presets
```

### Check system capabilities
```bash
aivideoenhancer status
```
Returns: GPU name, VRAM, available encoders, VSR upscale/RIFE availability, trial status.

### Create a diagnostic bundle
```bash
aivideoenhancer diagnose
```

## Presets

| Preset ID | What it does | Best for |
|-----------|-------------|----------|
| `old_video_restoration` | Stabilize medium + Denoise strong + Upscale 2x + Audio cleanup | VHS tapes, old camcorder footage, family videos |
| `surveillance_enhancement` | Denoise medium + Upscale 4x | Security camera footage, dash cams |
| `content_creation` | Upscale 2x + Interpolate x2 | YouTube uploads, social media content |
| `drone_action_cam` | Stabilize strong + Upscale 2x + Interpolate x2 | GoPro, DJI drone footage, shaky handheld |
| `animation_anime` | Denoise light + Upscale 4x | Anime, cartoons, animated content |
| `video_archival` | Stabilize light + Denoise medium + Upscale 2x | Long-term storage of any footage |
| `enhance_cleanup` | Denoise light + same-resolution cleanup | Compression artifacts, no resolution change |
| `smooth_motion` | Interpolate x2 only (no upscale) | Making motion fluid at 2x the frame rate |

## Individual Enhancement Options

### Upscaling (NVIDIA Video Super Resolution)
- `--upscale off` — no upscaling
- `--upscale enhance` — clean up at the same resolution (no size change)
- `--upscale x2` / `--upscale x3` / `--upscale x4` — 2x/3x/4x resolution

Requires an **NVIDIA RTX GPU** with a recent driver. No CPU fallback — on non-NVIDIA hardware, upscaling fails gracefully.

### Frame Interpolation (RIFE)
- `--interpolate off` — no interpolation
- `--interpolate x2` — double the frame rate (e.g., 30fps to 60fps)
- `--interpolate x4` — quadruple the frame rate

Requires a Vulkan-capable GPU (NVIDIA or AMD). Interpolation multiplies fps; there is no exact-fps target.

### Stabilization (FFmpeg vidstab)
- `--stabilize off|light|medium|strong` — strong may crop edges

### Denoising
- `--denoise off|light|medium|strong`
- `--denoise-method nlmeans|hqdn3d` (default: hqdn3d)

### Deinterlace & Sharpen
- `--deinterlace off|yadif|yadifbob|bwdif`
- `--sharpen off|light|medium|strong`

### Output Codec
- `--codec h264` — maximum compatibility
- `--codec h265` — best quality/size balance (default)
- `--codec av1` — maximum compression (slower encoding)
- `--codec vp9` — WebM-friendly
- `--crf 1-51` — quality (lower = better, default 18)

Encoding uses NVIDIA NVENC / Intel QSV / AMD AMF when available, CPU otherwise.

## Enhancement Pipeline

The stages run in this order:
1. Deinterlace (if enabled)
2. Stabilization (if enabled)
3. Denoising (if enabled)
4. Frame extraction (if upscale or interpolation enabled)
5. AI Upscaling via NVIDIA VSR (if enabled)
6. AI Frame Interpolation via RIFE (if enabled)
7. Sharpen (if enabled)
8. Final encode with target codec

Each stage is optional. Only enabled stages run.

## Exit Codes

0 success · 1 general error · 2 invalid arguments · 3 license required (trial expired — **`serve` only**) · 4 enhancement failed · 5 file not found · 6 tools missing

## Guidelines

- Always run `aivideoenhancer status` first to verify GPU and tool availability, then `analyze <file>` to understand the source video.
- Upscaling requires an NVIDIA RTX GPU. If `status` shows no VSR support, skip upscaling and use denoise/stabilize/sharpen instead.
- For old/degraded footage, start with the `old_video_restoration` preset.
- For shaky action cam/drone footage, use `drone_action_cam`.
- To clean up compression artifacts without changing resolution, use `enhance_cleanup` or `--upscale enhance`.
- Upscaling x4 on long videos is very slow — warn the user about processing time.
- Upscaling works best on clean source material. If the source is noisy, enable denoising too.
- The `animation_anime` preset uses the same VSR upscaler as the other presets (there is no separate anime model) — it just pairs light denoise with a 4x upscale.
- Trial: 30 days; the first 5 files (lifetime) are un-branded, then output gets a watermark and a 1280x720 cap until registered.
