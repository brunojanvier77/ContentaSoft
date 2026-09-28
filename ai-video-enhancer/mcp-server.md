# AI Video Enhancer Studio: MCP server

`aivideoenhancer serve` exposes 4 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Desktop, Claude Code, Cursor, Windsurf and others) can analyze and enhance videos on your PC.

## Setup

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

If your AI client cannot find `aivideoenhancer`, use the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\AIVideoEnhancerStudio\\aivideoenhancer.exe"`. See the [MCP config guide](../mcp-config/) for each client.

Paths can be absolute or relative. A relative path is resolved against the server's working folder, which is the folder your AI client started it in; use absolute paths when you do not know that folder. Remix and frame extraction are CLI-only (`aivideoenhancer remix`, `aivideoenhancer extract-frames`).

The server writes only JSON-RPC messages to stdout; its log lines go to stderr.

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio |
| Protocol | JSON-RPC 2.0, MCP `2024-11-05` |
| Server name | `ai-video-enhancer` |
| Server version | `2026.7.14` |

**License**: the server runs during the 30-day trial and for registered copies; after the trial it refuses to start (exit code 3). During the trial the first 5 files are full resolution without a watermark; later output carries a watermark and is capped at 1280x720.

## Tools

### analyze_video

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | Yes | Path to the video |

**Returns**: path, fileName, fileSize, fileSizeFormatted, videoCodec, videoCodecLong, resolution, width, height, frameRate, frameCount, duration (seconds), durationFormatted, videoBitrate, totalBitrate, bitrateFormatted, pixelFormat, containerFormat, isHdr, isInterlaced, audioCodec, audioChannels, audioSampleRate, audioBitrate.

---

### enhance_video

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `input_path` | string | Yes | Path to the video |
| `output_path` | string | No | Output file or folder (default: the input's folder) |
| `preset` | string | No | `old_video_restoration`, `surveillance_enhancement`, `content_creation`, `drone_action_cam`, `animation_anime`, `video_archival`, `enhance_cleanup`, `smooth_motion` |
| `upscale` | string | No | `off`, `enhance` (same resolution), `x2`, `x3`, `x4` |
| `denoise` | string | No | `off`, `light`, `medium`, `strong` |
| `stabilize` | string | No | `off`, `light`, `medium`, `strong` |
| `deinterlace` | string | No | `off`, `yadif`, `yadifbob`, `bwdif`. With a `preset` and no value, an interlaced source gets `yadif`; pass `off` to keep it off |
| `sharpen` | string | No | `off`, `light`, `medium`, `strong` |
| `rolling_shutter` | string | No | Jello correction: `off`, `light`, `medium`, `strong` |
| `interpolate` | string | No | Target frame rate: `off`, `30`, `60`. Only raises the rate |
| `codec` | string | No | `h264`, `h265`, `av1` |
| `crf` | integer | No | 0-51, lower = better (default 18) |
| `skip_existing` | boolean | No | Leave the video alone if its output already exists; a skipped video uses no free trial file |

Explicit parameters override the preset. Upscaling needs an NVIDIA RTX GPU; interpolation needs a Vulkan GPU (check with `get_status`).

**Returns**: success, inputPath, outputPaths; deinterlaceNotice when a preset switched deinterlacing on; errors when something failed. The output file is `<name>_enhanced.mp4`.

---

### list_presets

No parameters.

**Returns**: an array of presets with id, name, description, features, codec, crf. `features` is the full pipeline in order, for example `["Stabilize:Medium", "Denoise:Strong", "Upscale:X2", "AudioCleanup:Strong"]` for `old_video_restoration` and `["Denoise:Light", "Interpolate:Fps30"]` for `smooth_motion`.

---

### get_status

No parameters.

**Returns**:

| Field | Contents |
|-------|----------|
| `system` | os, processors |
| `gpu` | name, vendor, vramMb, vulkan, summary, recommendedTileSize, hwEncoders (h264/h265/av1/vp9) |
| `tools` | vsrUpscale: whether NVIDIA upscaling is available |
| `license` | registered, email; trialDaysRemaining while in trial |

## Example requests

> "Can my PC upscale video?"
>
> The agent calls `get_status` and checks `tools.vsrUpscale` and `gpu.vulkan`.

> "Restore C:\Videos\wedding-1998.avi."
>
> The agent calls `analyze_video`, then `enhance_video` with `preset: "old_video_restoration"`. If `isInterlaced` is true the preset deinterlaces with `yadif` on its own; the agent can pass `deinterlace: "bwdif"` to choose another mode.
