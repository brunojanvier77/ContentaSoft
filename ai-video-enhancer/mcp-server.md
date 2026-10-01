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

Paths can be absolute or relative. A relative path is resolved against the server's working folder, which is the folder your AI client started it in; use absolute paths when you do not know that folder. Remix, frame extraction and the free clips are CLI-only (`aivideoenhancer remix`, `aivideoenhancer extract-frames`, `aivideoenhancer enhance --clip`).

The server writes only JSON-RPC messages to stdout; its log lines go to stderr.

In Claude Code on Windows one command adds it, and nothing else needs installing because the client calls the installed `aivideoenhancer.exe` directly:

```powershell
claude mcp add ai-video-enhancer -- aivideoenhancer serve
```

## Protocol

| Property | Value |
|----------|-------|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `ai-video-enhancer` |
| Server version | `2026.7.16` |

All four ContentaSoft servers run on the same host, which behaves like this:

- **stdout carries JSON-RPC only.** Logs and anything else go to stderr, so a client can parse every line it reads.
- **`ping` is answered while a tool runs**, so a long enhancement does not look like a hung server.
- **`notifications/cancelled` cancels a running call.** The call stops and, as the specification says, gets no reply.
- **Tool calls run one at a time**, in the order they arrive. `ping`, `tools/list` and cancellations are handled in between.
- **Errors**: an unknown tool or method is a JSON-RPC error (`-32602`, `-32601`); a tool that cannot do its job returns a normal result with `isError: true` and a sentence that names the argument and what it accepts. A call missing a required argument says which one.
- **Every tool has a `title` and `annotations`** (`readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`); see the next section.

**Trial**: the server never refuses to start, and the trial has no end date. This computer's first 5 full exports (lifetime, plus 10 after the newsletter confirmation in the app) are full resolution without a watermark; later output carries a watermark and is capped at 1280x720. Register with `aivideoenhancer register <email> <key>`. The free 10-second clips (`--clip`) are a CLI feature; `enhance_video` always writes a full export.

## Tool annotations

| Tool | Title | Read-only | Destructive | Network |
|------|-------|:---------:|:-----------:|:-------:|
| `analyze_video` | Analyze video | yes | no | no |
| `enhance_video` | Enhance video | no | no | no |
| `list_presets` | List presets | yes | no | no |
| `get_status` | Get GPU and license status | yes | no | no |

`enhance_video` writes a new file (a taken name gets a renamed copy beside it, unless `skip_existing` is set) and never replaces the source. No tool uses the network. The three read-only tools are also marked idempotent.

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
| `output_path` | string | No | A file path ending `.mp4`, `.mov` or `.mkv`, or a folder (default: the input's folder). A relative path is resolved from the server's working folder |
| `preset` | string | No | `old_video_restoration`, `surveillance_enhancement`, `content_creation`, `drone_action_cam`, `animation_anime`, `video_archival`, `enhance_cleanup`, `smooth_motion` |
| `upscale` | string | No | `off`, `enhance` (same resolution), `x2`, `x3`, `x4` |
| `denoise` | string | No | `off`, `light`, `medium`, `strong` |
| `stabilize` | string | No | `off`, `light`, `medium`, `strong` |
| `deinterlace` | string | No | `off`, `yadif`, `yadifbob`, `bwdif`. With a `preset` and no value, an interlaced source gets `yadif`; pass `off` to keep it off |
| `sharpen` | string | No | `off`, `light`, `medium`, `strong` |
| `rolling_shutter` | string | No | Jello correction: `off`, `light`, `medium`, `strong` |
| `interpolate` | string | No | Target frame rate: `off`, `30`, `60`. Only raises the rate |
| `codec` | string | No | `h264`, `h265`, `av1`, `vp9` |
| `crf` | integer | No | 1-51, lower = better (default 18) |
| `skip_existing` | boolean | No | Leave the video alone if its output already exists; a skipped video uses no free trial file |

Explicit parameters override the preset. Upscaling needs an NVIDIA RTX GPU; interpolation needs a Vulkan GPU (check with `get_status`).

Arguments are checked before any work starts. A value outside an enumerated list (`upscale: "x9"`, `codec: "foo"`), a `crf` outside 1-51 and a non-integer number are errors that say which argument was wrong and what it accepts; none of them falls back to a default. A file with no video stream is refused with an error.

**Returns**: success, inputPath, outputPaths; `skipped: true` when `skip_existing` left the video alone; deinterlaceNotice when a preset switched deinterlacing on; errors when something failed (the result then has `isError: true`). With a file path in `output_path` the output is exactly that file; otherwise it is `<name>_enhanced.mp4` in the folder.

During the trial, once the free full exports are used up, the output carries the watermark and is capped at 1280x720, and the result also has `trialWatermarked: true`. While the trial lasts, the result has a `trialNotice` that says how many free files are left or that they are used up, with the buy link.

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
| `tools` | vsrUpscale: whether the NVIDIA upscaling helper is installed |
| `upscale` | Whether an `enhance_video` call with an upscale mode can run on this GPU: `available` (true or false), `state` (`available`, `unavailable` or `unknown`), `reason` and `note` when it cannot (`nvidia_without_rtx`, `non_nvidia_gpu`, `vsr_helper_missing`) |
| `license` | registered, email; while in trial, freeExportsRemaining and freeExportAllowance (the full exports left, and the total including any newsletter bonus) |

Read `upscale.available` before offering an upscale: `tools.vsrUpscale` only says the helper file exists, and on a machine without an NVIDIA RTX GPU every upscale job fails. Denoise, sharpen, stabilize and deinterlace still work there. `state: "unknown"` means the GPU could not be identified; upscale is not blocked.

## Example requests

> "Can my PC upscale video?"
>
> The agent calls `get_status` and checks `upscale.available` (upscaling) and `gpu.vulkan` (interpolation).

> "Restore C:\Videos\wedding-1998.avi."
>
> The agent calls `analyze_video`, then `enhance_video` with `preset: "old_video_restoration"`. If `isInterlaced` is true the preset deinterlaces with `yadif` on its own; the agent can pass `deinterlace: "bwdif"` to choose another mode.
