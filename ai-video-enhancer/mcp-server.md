# AI Video Enhancer Studio: MCP server

`aivideoenhancer serve` exposes 6 tools over the [Model Context Protocol](https://modelcontextprotocol.io/), so an AI agent (Claude Code, Claude Desktop, Cursor, Windsurf, Codex and others) can work on files on your PC.

Generated from `aivideoenhancer catalog`, version 2026.7.21. The same reference is on the website: [www.contenta-software.com/aivideoenhancer/docs/cli.html](https://www.contenta-software.com/aivideoenhancer/docs/cli.html). The command line is documented in [cli.md](cli.md).

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

If your AI client cannot find `aivideoenhancer`, give the full path, for example `"C:\\Users\\<you>\\AppData\\Local\\Programs\\AIVideoEnhancerStudio\\aivideoenhancer.exe"`. The [MCP config guide](../mcp-config/) covers each client. In Claude Code one command adds it:

```powershell
claude mcp add ai-video-enhancer -- aivideoenhancer serve
```

Pass absolute paths to every tool.

## Protocol

| Property | Value |
|---|---|
| Transport | stdio, newline-delimited JSON-RPC 2.0 |
| Protocol versions | `2025-11-25`, `2025-06-18`, `2025-03-26`, `2024-11-05`. The server answers with the version the client asks for; for any other version it answers `2025-11-25` |
| Capabilities | `tools` only (no resources or prompts) |
| Server name | `ai-video-enhancer` |
| Server version | `2026.7.21` |

All four ContentaSoft servers run on the same host, which behaves like this:

- **stdout carries JSON-RPC only.** Logs go to stderr, so a client can parse every line it reads.
- **`ping` is answered while a tool runs**, so a long job does not look like a hung server.
- **`notifications/cancelled` cancels a running call.** The call stops and, as the specification says, gets no reply.
- **Tool calls run one at a time**, in the order they arrive.
- **Errors**: an unknown tool or method is a JSON-RPC error; a tool that cannot do its job returns a normal result with `isError: true` and a sentence that names the problem.
- **Licence first**: `get_license_status` returns the licence and trial state, so an agent can tell the user what a run will produce before it starts one.

## Tool annotations

| Tool | Title | Read-only | Destructive | Idempotent | Network |
|---|---|---|---|---|---|
| [`enhance_video`](#enhance_video) | Enhance video | no | no | no | no |
| [`get_file_info`](#get_file_info) | Get video file info | yes | no | yes | no |
| [`get_license_status`](#get_license_status) | Get license and trial status | yes | no | yes | no |
| [`get_status`](#get_status) | Get GPU and license status | yes | no | yes | no |
| [`list_formats`](#list_formats) | List formats | yes | no | yes | no |
| [`list_presets`](#list_presets) | List presets | yes | no | yes | no |

Destructive: the tool can replace a file that already exists. Network: the tool uses the internet.

## Tools

### enhance_video

**Enhance video.** Enhance a video using AI upscaling, frame interpolation, stabilization, and denoising.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the input video file. |
| `output_path` | string | no | Output file path (ending .mp4, .mov or .mkv) or output directory; a relative path is resolved from the server's working directory (default: same directory as input). |
| `preset` | string | no | Preset ID: tape_restoration (VHS, Video8, Hi8 and MiniDV captures: full-motion deinterlace to 50p/59.94p, colour noise and hum removal, square-pixel 4:3; CPU only, no graphics card needed), old_video_restoration, surveillance_enhancement, content_creation, drone_action_cam, animation_anime, video_archival, enhance_cleanup, smooth_motion. list_presets says what each one needs from the PC. |
| `upscale` | string | no | Upscale mode: off, enhance (same-resolution cleanup), x2, x3, x4. One of: `off`, `enhance`, `x2`, `x3`, `x4`. |
| `upscale_engine` | string | no | What upscales x2, x3, x4: auto (default: NVIDIA RTX Video Super Resolution where it can run, otherwise the standard upscaler), rtx, standard (Lanczos resize + sharpening on the CPU, not AI; any PC). The result carries upscaleEngine. One of: `auto`, `rtx`, `standard`. |
| `denoise` | string | no | Denoise level: off, light, medium, strong. One of: `off`, `light`, `medium`, `strong`. |
| `stabilize` | string | no | Stabilization level: off, light, medium, strong. One of: `off`, `light`, `medium`, `strong`. |
| `stabilize_keep` | integer | no | With stabilize (or a preset that stabilizes): keep at least this percent of the picture, 50-100. Stabilizing zooms in to hide the moving edge, by as much as the shake needs; a limit stops the zoom there and leaves the part of the movement it has no room for. 100 = no zoom at all, the edge of the picture then moves. Leave it out to let the stabilizer choose. Range: 50 to 100. |
| `deinterlace` | string | no | Deinterlace: off, yadif, yadifbob, bwdif, bwdifbob (full motion: one frame per field, 25i becomes 50p). With a preset and no deinterlace value, an interlaced source (DV, DVD, TV capture) gets yadif automatically; pass off to keep it off. One of: `off`, `yadif`, `yadifbob`, `bwdif`, `bwdifbob`. |
| `denoise_method` | string | no | Denoise method: hqdn3d (default), nlmeans, chroma (colour noise reduced much harder than luma, for tape). One of: `hqdn3d`, `nlmeans`, `chroma`. |
| `hum` | string | no | Remove mains hum from the audio: off, auto (50 Hz for a 25/50 fps picture, 60 Hz for 29.97/59.94), 50, 60. Used with a preset or another enhancement; the audio is re-encoded. One of: `off`, `auto`, `50`, `60`. |
| `crop_bottom` | boolean | no | With preset tape_restoration: crop the head-switching noise band at the bottom of a VHS/Video8/Hi8 capture (12 lines of 576, 8 of 480); the picture keeps its proportions. |
| `sharpen` | string | no | Sharpen: off, light, medium, strong. One of: `off`, `light`, `medium`, `strong`. |
| `rolling_shutter` | string | no | Rolling-shutter (jello) correction: off, light, medium, strong. One of: `off`, `light`, `medium`, `strong`. |
| `interpolate` | string | no | Target frame rate for smoother motion (RIFE): off, 30, 60. Only raises the rate — a clip already at/above the target is left unchanged. One of: `off`, `30`, `60`. |
| `codec` | string | no | Output codec: h264, h265, av1, vp9. One of: `h264`, `h265`, `av1`, `vp9`. |
| `hdr` | string | no | HDR10 / HLG sources: keep (default) or sdr. keep = the output stays 10-bit HDR when the job has no AI stage (upscale, interpolate), no stabilize, no rolling_shutter and no nlmeans denoise, and the codec is h265, av1 or vp9; otherwise the picture is tone-mapped to SDR. sdr = always tone-map to SDR (BT.709). hdrNotice in the result says what was done. One of: `keep`, `sdr`. |
| `crf` | integer | no | Quality (CRF value, 1-51, lower = better quality, default: 18). Range: 1 to 51. |
| `skip_existing` | boolean | no | Leave the video alone when its output file already exists (default: write a renamed copy beside it). A skipped video is not enhanced and costs no free file. |
| `overwrite` | boolean | no | Replace the output file when it already exists (default: write a renamed copy beside it). Not with skip_existing. |
| `clip` | string | no | Export only a 10-second clip starting at this time (seconds such as 42 or 42.5, or m:ss / h:mm:ss such as 1:05) instead of the whole video, named \<name\>-clip; the result carries clipPath. On the trial a clip is clean and full resolution and does not use a free export; clips from one video can cover up to 30 seconds a day. |
| `compare` | boolean | no | Also write a side-by-side before/after video (source and enhanced, labelled) next to each output as \<name\>\_compare.mp4; the result carries comparePaths. |
| `compare_labels` | string | no | Custom labels for compare, as "FIRST\|SECOND" (default: "BEFORE\|AFTER"). |
| `compare_seconds` | number | no | Length of the compare video in seconds, taken a third of the way into the video (default 0: the whole video; 30 gives a short clip to show a customer). Range: 0 to 3600. |
| `compare_layout` | string | no | Layout for compare: horizontal (default, side by side) or vertical (stacked top/bottom, avoids the very wide 32:9 frame two 16:9 panels make side by side). One of: `horizontal`, `vertical`. |

### get_file_info

**Get video file info.** Get a video file's metadata: codec, resolution, duration, bitrate, frame rate, audio, HDR (and what enhance_video will do with the HDR).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_path` | string | yes | Absolute path to the video file. |

### get_license_status

**Get license and trial status.** Get the license and trial state before enhancing: registered or trial, free full exports left (freeFilesRemaining), and what exports become after the free ones (afterFreeFiles: watermark_720p = a watermark and output capped at 1280x720; 10-second clips are never branded). nextOutput says what the next full export will be: clean or watermark_720p.

No parameters.

### get_status

**Get GPU and license status.** Get system status including GPU capabilities, available tools, and license state.

No parameters.

### list_formats

**List formats.** List the video containers AI Video Enhancer reads and writes, the output codecs (enhance_video codec) and the still formats of frame extraction: format, extensions, canImport, canExport, category.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `input_only` | boolean | no | Only what is read (the CLI's --input). |
| `output_only` | boolean | no | Only what is written (the CLI's --output). |

### list_presets

**List presets.** List available video enhancement presets.

No parameters.
