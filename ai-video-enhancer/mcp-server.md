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

## Protocol Details

| Property | Value |
|----------|-------|
| Transport | stdio (stdin/stdout) |
| Protocol | JSON-RPC 2.0 |
| Protocol Version | `2024-11-05` |
| Server Name | `ai-video-enhancer` |
| Server Version | `2026.1.0` |

## Tools

### analyze_video

Analyze a video file and return codec, resolution, duration, bitrate, frame rate, and audio information.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Absolute path to the video file |

**Returns**: fileName, fileSize, videoCodec, resolution, width, height, frameRate, frameCount, duration, videoBitrate, pixelFormat, colorSpace, isHdr, isInterlaced, audioCodec, audioChannels, containerFormat.

---

### enhance_video

Enhance a video using AI upscaling, frame interpolation, stabilization, and denoising.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Absolute path to the source video |
| `output_path` | string | No | Output file path (default: auto-generated) |
| `preset` | string | No | Preset ID (see list below) |
| `upscale` | string | No | `off`, `x2`, `x4` (default: off) |
| `denoise` | string | No | `off`, `light`, `medium`, `strong` (default: off) |
| `stabilize` | string | No | `off`, `light`, `medium`, `strong` (default: off) |
| `interpolate` | string | No | `off`, `double`, `to60fps` (default: off) |
| `codec` | string | No | Output codec: `h264`, `h265`, `av1` (default: h265) |
| `crf` | integer | No | Quality 0-51 (lower = better, default: 18) |

Individual parameters override preset values when both are provided.

**Returns**: success, outputPath, errors.

---

### list_presets

List available video enhancement presets. No parameters required.

**Returns**:

| Preset ID | Focus | Best For |
|----|-------|----------|
| `old_video_restoration` | Denoise + Upscale 2x + Stabilize | Old/degraded footage |
| `surveillance_enhancement` | Denoise + Upscale 4x | Security cameras |
| `content_creation` | Upscale 2x + Interpolate 60fps | YouTube/social |
| `drone_action_cam` | Strong Stabilization + Denoise | Action/drone footage |
| `animation_anime` | Upscale 4x (anime model) + Interpolate 60fps | Animated content |
| `video_archival` | Stabilize + Denoise + Upscale 2x | Long-term storage |

---

### get_status

Get system status including GPU capabilities, available tools, and license state.

No parameters required.

**Returns**: gpuName, gpuVendor, vramMb, hasVulkan, availableEncoders, realesrganAvailable, rifeAvailable, licenseState, trialDaysRemaining.
