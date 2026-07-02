# AI Video Enhancer Studio — MCP Server

AI Video Enhancer Studio exposes 4 tools for AI-powered video enhancement via the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP).

## Getting Started

```json
{
  "mcpServers": {
    "ai-video-enhancer": {
      "command": "aivideoenhancer",
      "args": ["serve"]
    }
  }
}
```

If `aivideoenhancer` is not on your `PATH`, use the full path: `C:\Program Files\ContentaSoft\AI Video Enhancer Studio\aivideoenhancer.exe`.

The server works during an active trial and is blocked once the trial expires (exit code 3, `LicenseRequired`).

## Protocol Details

| Property | Value |
|----------|-------|
| Transport | stdio (stdin/stdout) |
| Protocol | JSON-RPC 2.0 |
| Protocol Version | `2024-11-05` |
| Server Name | `ai-video-enhancer` |
| Server Version | `2026.3.4` |

## Tools

### analyze_video

Analyze a video file and return codec, resolution, duration, bitrate, frame rate, and audio information.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video file |

**Returns**: path, fileName, fileSize, fileSizeFormatted, videoCodec, videoCodecLong, resolution, width, height, frameRate, frameCount, duration (seconds), durationFormatted, videoBitrate, totalBitrate, bitrateFormatted, pixelFormat, containerFormat, isHdr, isInterlaced, audioCodec, audioChannels, audioSampleRate, audioBitrate, subtitleStreams.

---

### enhance_video

Enhance a video using AI upscaling, stabilization, and denoising.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source video |
| `output_path` | string | No | Output **directory** (default: same directory as input) |
| `preset` | string | No | Preset ID (see list below) |
| `upscale` | string | No | `off`, `enhance` (same-resolution cleanup), `x2`, `x3`, `x4` (default: off) |
| `denoise` | string | No | `off`, `light`, `medium`, `strong` (default: off) |
| `stabilize` | string | No | `off`, `light`, `medium`, `strong` (default: off) |
| `codec` | string | No | Output codec: `h264`, `h265`, `av1` (default: h265) |
| `crf` | integer | No | Quality 0-51 (lower = better, default: 18) |

Individual parameters override preset values when both are provided. There is no direct `interpolate` parameter — to get RIFE frame interpolation via MCP, use a preset that includes it (`content_creation`, `drone_action_cam`, `smooth_motion`), or use the CLI `--interpolate` flag.

**Returns**: success, inputPath, outputPaths, errors.

---

### list_presets

List available video enhancement presets. No parameters required.

**Returns** (per preset): id, name, description, features, codec, crf.

| Preset ID | Focus | Best For |
|----|-------|----------|
| `old_video_restoration` | Stabilize medium + Denoise strong + Upscale 2x + Audio cleanup | Old VHS/DVD/analog footage |
| `surveillance_enhancement` | Denoise medium + Upscale 4x | Security cameras |
| `content_creation` | Denoise light + Upscale 2x + Interpolate x2 | YouTube/social |
| `drone_action_cam` | Stabilize strong + Upscale 2x + Interpolate x2 | Action/drone footage |
| `animation_anime` | Denoise light + Upscale 4x | Animated content |
| `video_archival` | Stabilize light + Denoise medium + Upscale 2x | Long-term storage |
| `enhance_cleanup` | Denoise light + same-resolution VSR cleanup | Artifact cleanup, no resolution change |
| `smooth_motion` | Denoise light + Interpolate x2 (no upscale) | Fluid motion at 2x the frame rate |

All presets use the same NVIDIA VSR upscaler — there is no separate anime model. Interpolation multiplies the source frame rate (x2); no preset targets an exact fps.

---

### get_status

Get system status including GPU capabilities, available tools, and license state.

No parameters required.

**Returns**:

| Field | Contents |
|-------|----------|
| `system` | os, processors |
| `gpu` | name, vendor, vramMb, vulkan, summary, recommendedTileSize, hwEncoders (h264/h265/av1/vp9, or null if no hardware encoder) |
| `tools` | vsrUpscale (whether the NVIDIA VSR upscale helper is available) |
| `license` | registered, email, trialDaysRemaining (null when registered) |
