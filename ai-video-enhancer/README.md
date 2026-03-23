# AI Video Enhancer Studio

AI-powered video enhancement for Windows. Upscale, interpolate frames, stabilize, and denoise videos using Real-ESRGAN and RIFE.

[Download Free Trial](https://contenta-software.com)

## Features

- **AI upscaling** — Real-ESRGAN 2x/4x super-resolution (Vulkan GPU)
- **Frame interpolation** — RIFE AI to increase frame rate (30fps to 60fps, etc.)
- **Stabilization** — FFmpeg vidstab for shaky footage
- **Denoising** — nlmeans/hqdn3d noise reduction
- **GPU acceleration** — NVIDIA NVENC, Intel QSV, AMD AMF for encoding
- **6 presets** — old video restoration, surveillance, content creation, drone/action cam, animation, archival

## CLI Reference

**Executable**: `aivideoenhancer`

| Command | Description |
|---------|-------------|
| `enhance` | Enhance a video file |
| `analyze` | Analyze video properties |
| `presets` | List available enhancement presets |
| `status` | Check trial/registration status |

## Presets

| ID | Focus | Best For |
|----|-------|----------|
| `old_video_restoration` | Denoise + Upscale + Stabilize | Old/degraded footage |
| `surveillance_enhancement` | Denoise + Upscale | Security cameras |
| `content_creation` | Upscale + Interpolation | YouTube/social |
| `drone_action_cam` | Strong Stabilization + Denoise | Action/drone footage |
| `animation_anime` | Upscale (anime model) + Interpolation | Animated content |
| `video_archival` | Denoise + Encode (H.265) | Long-term storage |

## CLI Example

```bash
# Enhance old footage
aivideoenhancer enhance old-video.mp4 --output enhanced.mp4 --preset old_video_restoration
```
