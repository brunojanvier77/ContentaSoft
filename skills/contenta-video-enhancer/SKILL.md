---
name: contenta-video-enhancer
description: Enhance videos using the aivideoenhancer CLI with AI upscaling (Real-ESRGAN), frame interpolation (RIFE), stabilization, and denoising. Use when the user asks to upscale video resolution, increase frame rate, fix shaky footage, remove noise, restore old video, or improve video quality.
allowed-tools: Bash
---

# Contenta AI Video Enhancer

You have access to the `aivideoenhancer` CLI for AI-powered video enhancement.

## Commands

### Enhance a video
```bash
aivideoenhancer enhance <input> --output <file> --preset <preset_id>
```
Or with individual settings:
```bash
aivideoenhancer enhance <input> --output <file> --upscale x2 --denoise medium --stabilize light --codec h265 --crf 18
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
Returns: GPU name, VRAM, available encoders, Real-ESRGAN/RIFE availability.

## Presets

| Preset ID | What it does | Best for |
|-----------|-------------|----------|
| `old_video_restoration` | Denoise + Upscale 2x + Stabilize medium | VHS tapes, old camcorder footage, family videos |
| `surveillance_enhancement` | Denoise strong + Upscale 4x | Security camera footage, dash cams |
| `content_creation` | Upscale 2x + Interpolate to 60fps | YouTube uploads, social media content |
| `drone_action_cam` | Stabilize strong + Denoise | GoPro, DJI drone footage, shaky handheld |
| `animation_anime` | Upscale 4x (anime model) + Interpolate 60fps | Anime, cartoons, animated content |
| `video_archival` | Stabilize light + Denoise + Upscale 2x | Long-term storage of any footage |

## Individual Enhancement Options

### Upscaling (Real-ESRGAN)
- `--upscale off` — no upscaling
- `--upscale x2` — 2x resolution (e.g., 1080p to 4K)
- `--upscale x4` — 4x resolution (e.g., 480p to 1080p)

Uses Vulkan GPU acceleration. Falls back to CPU if no compatible GPU.

### Frame Interpolation (RIFE)
- `--interpolate off` — no interpolation
- `--interpolate double` — double the frame rate (e.g., 30fps to 60fps)
- `--interpolate to60fps` — target 60fps regardless of source

### Stabilization (FFmpeg vidstab)
- `--stabilize off` — no stabilization
- `--stabilize light` — subtle correction
- `--stabilize medium` — moderate correction
- `--stabilize strong` — aggressive correction (may crop edges)

### Denoising
- `--denoise off` — no denoising
- `--denoise light` — subtle noise removal
- `--denoise medium` — moderate (good for indoor/low-light)
- `--denoise strong` — aggressive (good for very noisy sources)

### Output Codec
- `--codec h264` — maximum compatibility
- `--codec h265` — best quality/size balance (default)
- `--codec av1` — maximum compression (slower encoding)

## Enhancement Pipeline

The stages run in this order:
1. Stabilization (if enabled)
2. Denoising (if enabled)
3. Frame extraction (if upscale or interpolation needed)
4. AI Upscaling via Real-ESRGAN (if enabled)
5. AI Frame Interpolation via RIFE (if enabled)
6. Final encode with target codec

Each stage is optional. Only enabled stages run.

## Guidelines

- Always run `aivideoenhancer analyze <file>` first to understand the source video.
- Always run `aivideoenhancer status` first to verify GPU and tool availability.
- For old/degraded footage, start with `old_video_restoration` preset.
- For shaky action cam/drone footage, use `drone_action_cam` preset.
- Upscaling x4 on long videos is very slow — warn the user about processing time.
- Upscaling works best on clean source material. If the source is noisy, enable denoising too.
- Frame interpolation to 60fps is most noticeable on footage originally shot at 24-30fps.
- The `animation_anime` preset uses a specialized anime model that produces better results on drawn/animated content than the general model.
- GPU with Vulkan support is strongly recommended for upscaling and interpolation. Check with `aivideoenhancer status`.
- Trial users can enhance freely for 30 days (watermark after 10 files).
